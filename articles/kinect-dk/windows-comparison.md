---
title: Comparaison d’Azure Kinect DK Windows
description: Différences matérielles et logicielles entre Azure Kinect DK et Kinect pour Windows v2
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: article
keywords: Kinect, Windows, V2, Azure Kinect, comparaison, Kit de développement logiciel (SDK), différences, matériel, logiciels
ms.openlocfilehash: 0a8d399370f354524858bdd658ffd65c0494dd4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87031573"
---
# <a name="azure-kinect-and-kinect-windows-v2-comparison"></a>Comparaison entre Azure Kinect et Kinect pour Windows v2

Le matériel et les kits de développement logiciel (SDK) Azure Kinect DK présentent des différences par rapport à Kinect pour Windows v2. Les applications Kinect pour Windows v2 existantes ne fonctionnent pas directement avec Azure Kinect DK et nécessitent un portage vers le nouveau Kit de développement logiciel (SDK).  

## <a name="hardware"></a>Matériel

Les grandes différences entre le kit de développement Azure Kinect et Kinect pour Windows v2 sont répertoriées dans le tableau suivant.

| Fonctionnalité | Type | Azure Kinect DK | Kinect pour Windows v2 |
| ------- | ---- | --------------- | --------------------- |
| **Audio** | Détails  | Tableau circulaire pour 7 micros | Tableau linéaire pour 4 micros |
| **Capteur de mouvement** | Détails | Accéléromètre sur 3 axes Gyroscope sur 3 axes | Accéléromètre sur 3 axes |
| **Caméra RVB**    | Détails | 3840 x 2160 px @30 i/s | 1920 x 1080 px @30 i/s |
| **Caméra de profondeur**  | Méthode   | Temps de vol | Temps de vol |
|                   | Résolution | 640 x 576 px @30 i/s | 512 x 424 px @ 30 i/s |
|                   |            | 512 x 512 px @30 i/s |                       |
|                   |            | 1024 x 1024 px @15 i/s |                       |
| **Connectivité** | Données | USB 3.1 Gen 1 avec type USB-C  | USB 3.1 Gen 1|
|  | Power | Bloc d’alimentation ou USB-C externes | Bloc d’alimentation externe |
|  | Synchronization | Connexion appareil à appareil interne, externe, RVB et profondeur| Interne RVB et profondeur uniquement |
| **Mécanique** | Dimensions | 103 x 39 x 126mm | 249 x 66 x 67 mm |
|  | Masse | 440 g | 970 g |
| | Montage | Un UNC ¼-20 Quatre points de vissage internes | Un UNC ¼-20 |

Pour plus d’informations, consultez le document [Matériel Azure Kinect DK](hardware-specification.md).

## <a name="sensor-access"></a>Accès au capteur

Le tableau suivant fournit une comparaison des fonctionnalités d’accès aux capteurs.

| **Fonctionnalité**| **Azure Kinect** | **Kinect pour Windows** | **Remarques** |
|---------|---------|------------|---------|
| **Profondeur** | ✔️ | ✔️ |    |   |
| **IR** | ✔️ | ✔️ |  |
| **Color** | ✔️ | ✔️ | Le format des couleurs prend en charge les différences, Azure Kinect DK prend en charge les contrôles de caméra suivants : exposition, balance des blancs, luminosité, contraste, saturation, netteté et réglage de gain |
| **Audio** | ✔️ | ✔️ | Les micros Azure Kinect DK sont accessibles via le Kit de développement logiciel (SDK) Speech ou une API native Windows |
| **IMU** | ✔️ |  | Azure Kinect DK offre une IMU sur 6 axes et Kinect pour Windows uniquement sur un 1 axe |
| **Données d’étalonnage** | ✔️ | ✔️ | Étalonnage du modèle de caméra compatible OpenCV |
| **Synchronisation interne Profondeur-RVB** | ✔️ | ✔️ |  |
| **Synchronisation externe**| ✔️|  | Azure Kinect DK autorise un délai programmable pour la synchronisation externe |
| **Partager l’accès avec plusieurs clients** | | ✔️ | Le Kit de développement logiciel (SDK) du capteur Azure Kinect s’appuie sur WinUSB/libUSB pour accéder à l’appareil et n’a pas de service implémenté pour permettre le partage d’accès à l’appareil entre plusieurs processus |
| **Outil d’enregistrement/de lecture de flux** | ✔️ | ✔️ | Azure Kinect DK utilise une implémentation basée sur un conteneur Matroska open source |

## <a name="features"></a>Fonctionnalités

Le jeu de fonctionnalités du Kit de développement logiciel (SDK) Azure Kinect est différent de Kinect pour Windows v2, comme indiqué ci-dessous :

| **Fonctionnalité Kinect v2** | **Type de données Kinect v2** | **Service/Kit de développement logiciel (SDK) Azure Kinect** |
|--------|--------|------|
| Accès aux données du capteur |DepthFrame| [Kit de développement logiciel (SDK) de capteur – Récupérer des images](retrieve-images.md) 
| |InfraredFrame | [Kit de développement logiciel (SDK) de capteur – Récupérer des images](retrieve-images.md) 
| | ColorFrame | [Kit de développement logiciel (SDK) de capteur – Récupérer des images](retrieve-images.md) | 
| | AudioBeamFrame |Non prise en charge pour le moment 
| Suivi de corps | BodyFrame | Kit de développement logiciel (SDK) Body Tracking |
| | BodyIndexFrame | Kit de développement logiciel (SDK) Body Tracking  |
| Mappage de coordonnées|CoordinateMapper| [Kit de développement logiciel (SDK) de capteur – Transformations d’images](use-image-transformation.md) |
|Suivi du visage | FaceFrame | [Cognitive Services: Face](https://azure.microsoft.com/services/cognitive-services/face/)       |
|    Reconnaissance vocale    |    N/A                      |    [Cognitive Services: Speech](https://azure.microsoft.com/services/cognitive-services/directory/speech/)     |

## <a name="next-steps"></a>Étapes suivantes

[Pages pour les développeurs Kinect pour Windows](https://developer.microsoft.com/windows/kinect)
