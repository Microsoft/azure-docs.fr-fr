---
title: Visionneuse Azure Kinect
description: Découvrez comment visualiser tous les flux de données d’appareil à l’aide de la visionneuse Azure Kinect.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: azure, kinect, capteur, visionneuse, visualisation, profondeur, rvb, couleur, imu, audio, microphone, nuage de points
ms.openlocfilehash: f411de5854d516586dc64701cca729c692ce40e0
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92166103"
---
# <a name="azure-kinect-viewer"></a>Visionneuse Azure Kinect

La visionneuse Azure Kinect qui se trouve dans le répertoire des outils installés comme `k4aviewer.exe` (par exemple, `C:\Program Files\Azure Kinect SDK vX.Y.Z\tools\k4aviewer.exe`, où `X.Y.Z` est la version installée du Kit de développement logiciel (SDK)), permet de visualiser tous les flux de données d’appareils aux fins suivantes :

* Vérifier que les capteurs fonctionnent correctement.
* Positionner l’appareil.
* Expérimenter des paramètres de caméra.
* Lire une configuration d’appareil.
* Lire des enregistrements effectués avec l’[Enregistreur Azure Kinect](azure-kinect-recorder.md).

Pour plus d’informations sur la Visionneuse Azure Kinect, regardez la [vidéo Comment utiliser Azure Kinect](https://www.microsoft.com/videoplayer/embed/RE3hNwG).

La visionneuse Azure Kinect est disponible en [open source](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/tools/k4aviewer) et peut être utilisée en tant qu’exemple pour l’utilisation des API.

## <a name="use-viewer"></a>Utiliser la visionneuse

La visionneuse peut fonctionner en deux modes : avec des données actives du capteur ou à partir de données enregistrées ([Enregistreur Azure Kinect](azure-kinect-recorder.md)).

### <a name="start-application"></a>Démarrer l’application

Lancez l’application en exécutant `k4aviewer.exe`.

![Capture d’écran montrant l’application de visionneuse lancée.](./media/how-to-guides/open-viewer.png)

### <a name="use-the-viewer-with-live-data"></a>Utiliser la visionneuse avec des données actives

1. Dans la section **Open Device** (Ouvrir un appareil), sélectionnez le **Serial Number** (Numéro de série) de l’appareil à ouvrir. Ensuite, sélectionnez **Refresh** (Actualiser) si l’appareil est manquant.
2. Sélectionnez le bouton **Open Device** (Ouvrir un appareil).
3. Sélectionnez **Start** (Démarrer) pour commencer la diffusion en continu des données avec les paramètres par défaut.

### <a name="use-the-viewer-with-recorded-data"></a>Utiliser la visionneuse avec des données enregistrées

Dans la section **Open Recording** (Ouvrir un enregistrement), accédez au fichier enregistré et sélectionnez-le.

## <a name="check-device-firmware-version"></a>Vérifier la version du microprogramme de l’appareil

Accédez à la version du microprogramme de l’appareil dans la fenêtre de configuration, comme illustré dans l’image suivante.

![Vérification de la version du microprogramme de l’appareil avec la visionneuse](./media/how-to-guides/check-firmware-update.png)

Par exemple, dans ce cas, le fournisseur d’accès Internet de la caméra de profondeur exécute FW 1.5.63.

## <a name="depth-camera"></a>Caméra à profondeur de champ

La visionneuse de la caméra de profondeur affiche deux fenêtres :

* L’une est appelée *Active Brightness* (Luminosité active), qui présente une image en nuances de gris montrant la luminosité de l’IR.
* L’autre est appelée *Depth* (Profondeur), qui présente une représentation en couleurs des données de profondeur.

Pointez le curseur sur le pixel de la fenêtre de profondeur pour afficher la valeur du capteur de profondeur, comme illustré ci-dessous.

![Affichage de profondeur](./media/how-to-guides/depth-camera.png)

## <a name="rgb-camera"></a>Caméra RVB

L’image ci-dessous montre l’affichage de la caméra couleur.

![Affichage RVB](./media/how-to-guides/viewer-rgb-camera.png)

Vous pouvez contrôler les paramètres de caméra RVB à partir de la fenêtre de configuration pendant la diffusion en continu.

![Contrôles de camera RVB](./media/how-to-guides/rgb-camera-settings.png)

## <a name="inertial-measurement-unit-imu"></a>Unité de mesure inertielle (IMU)

La fenêtre IMU affiche deux composants : un accéléromètre et un gyroscope.

La moitié supérieure correspond à l’accéléromètre et montre l’accélération linéaire en mètres/seconde<sup>2</sup>.  Elle comprend l’accélération à partir de la gravité. Ainsi, mis à plat sur une table, l’axe Z affichera probablement une valeur avoisinant -9,8 m/s<sup>2</sup>.

La moitié inférieure correspond au gyroscope et montre le mouvement de rotation en radians/seconde.

![Affichage du capteur de mouvement](./media/how-to-guides/viewer-mu-settings.png)

## <a name="microphone-input"></a>Entrée microphone

L’affichage du microphone représente le son audible sur chaque microphone. S’il n’y a pas de son, le graphique affiché est vide. Autrement, vous voyez une forme d’onde de couleur bleue foncée à laquelle est superposée une forme d’onde de couleur bleue claire.

L’onde foncée représente les valeurs minimales et maximales observées par le microphone pendant cette tranche horaire. L’onde claire représente la moyenne quadratique des valeurs observées par le microphone pendant cette tranche horaire.

![Vue d’entrée du microphone](./media/how-to-guides/microphone-data.png)

## <a name="point-cloud-visualization"></a>Visualisation de nuage de points

La profondeur visualisée en 3D vous permet de vous déplacer dans l’image à l’aide des touches.

![Nuage de points de profondeur](./media/how-to-guides/depth-point-cloud.png)

## <a name="synchronization-control"></a>Contrôle de synchronisation

Vous pouvez utiliser la visionneuse pour configurer l’appareil en mode autonome (par défaut), maître ou subordonné lors de la configuration de la synchronisation de plusieurs appareils.
Lors de la modification de la configuration ou de l’insertion ou du retrait du câble de synchronisation, sélectionnez **Actualiser** pour mettre à jour.

![Contrôle de synchronisation externe](./media/how-to-guides/sync-control.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Guide de configuration de la synchronisation externe](https://support.microsoft.com/help/4494429/sync-multiple-azure-kinect-dk-devices)
