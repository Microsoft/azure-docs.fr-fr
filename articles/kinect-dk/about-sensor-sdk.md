---
title: À propos du Kit de développement logiciel (SDK) du capteur Azure Kinect
description: Vue d’ensemble du Kit de développement logiciel (SDK) du capteur Azure Kinect, de ses fonctionnalités et de ses outils.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: article
keywords: azure, kinect, rvb, ir, enregistrement, capteur, sdk, accès, profondeur, vidéo, caméra, imu, mouvement, capteur, audio, microphone, matroska, kit de développement logiciel (sdk) de capteur, téléchargement
ms.openlocfilehash: 17c1b33120eacb5d0c6d3c02e692d1488ef474e6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276507"
---
# <a name="about-azure-kinect-sensor-sdk"></a>À propos du Kit de développement logiciel (SDK) du capteur Azure Kinect

Cet article fournit une vue d’ensemble du Kit de développement logiciel (SDK) du capteur Azure Kinect, de ses fonctionnalités et de ses outils.

## <a name="features"></a>Fonctionnalités

Le Kit de développement logiciel (SDK) du capteur Azure Kinect fournit un accès de bas niveau multiplateforme pour la configuration d’appareil Azure Kinect et les flux de capteurs matériels, à savoir :

- Accès aux caméras de profondeur et contrôle du mode (mode IR passif et modes de profondeur à champ de vision large et étroit) 
- Accès aux caméras RVB et contrôle de celles-ci (par exemple, exposition et balance des blancs) 
- Accès aux capteurs de mouvement (gyroscope et accéléromètre) 
- Diffusion synchronisée des données des caméras RVB de profondeur avec la possibilité de configurer le délai entre les caméras 
- Contrôle de synchronisation des appareils externes avec la possibilité de configurer le décalage entre les appareils 
- Accès aux métadonnées de vue de caméra pour la résolution d’image, l’horodatage, etc. 
- Accès aux données d’étalonnage des appareils 

## <a name="tools"></a>Outils

- Une [visionneuse Azure Kinect](azure-kinect-viewer.md) permettant de superviser les flux de données des appareils et de configurer les différents modes.
- Un [Enregistreur Azure Kinect](azure-kinect-recorder.md) et une API de lecture de capteur utilisant le [format de conteneur Matroska](record-file-format.md).
- Un [outil de mise à jour de microprogramme](azure-kinect-firmware-tool.md) Azure Kinect DK.

## <a name="sensor-sdk"></a>Kit de développement logiciel (SDK) du capteur

- [Téléchargez le Kit de développement logiciel (SDK) du capteur](sensor-sdk-download.md).
- Le Kit de développement logiciel (SDK) du capteur est disponible en [open source sur GitHub](https://github.com/microsoft/Azure-Kinect-Sensor-SDK).
- Pour plus d’informations sur l’utilisation, consultez la [documentation de l’API du Kit de développement logiciel (SDK) du capteur](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/index.html).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert le kit de développement logiciel (SDK) du capteur Azure Kinect, vous pouvez également :
>[!div class="nextstepaction"]
>[Télécharger le code du Kit de développement logiciel (SDK) du capteur](sensor-sdk-download.md)

>[!div class="nextstepaction"]
>[Rechercher et ouvrir un appareil](find-then-open-device.md)
