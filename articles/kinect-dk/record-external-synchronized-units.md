---
title: Utiliser l’enregistreur Kinect Azure avec des appareils externes synchronisés
description: Découvrez comment enregistrer des données à partir d’appareils configurés pour une synchronisation externe à l’aide de l’Enregistreur Azure Kinect.
author: tesych
ms.author: tesych
ms.reviewer: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, capteur, visionneuse, synchronisation externe, retard de phase, profondeur, RVB, caméra, câble audio, enregistreur
ms.openlocfilehash: 052f6f1ac9f90e764de25d1d4d1b25b3d50a848d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276460"
---
# <a name="use-azure-kinect-recorder-with-external-synchronized-devices"></a>Utiliser l’enregistreur Kinect Azure avec des appareils externes synchronisés

Cet article fournit des conseils sur la façon dont l’[Enregistreur Azure Kinect](azure-kinect-recorder.md) peut enregistrer des appareils configurés pour une synchronisation externe de données.

## <a name="prerequisites"></a>Prérequis

- [Configurez plusieurs unités Azure Kinect DK pour la synchronisation externe](https://support.microsoft.com/help/4494429).

## <a name="external-synchronization-constraints"></a>Contraintes de synchronisation externe

- L’appareil maître ne peut pas avoir de câble SYNC IN connecté.
- L’appareil maître doit diffuser la caméra RVB pour permettre la synchronisation.
- Toutes les unités doivent utiliser la même configuration de caméra (taux de trames et résolution).
- Toutes les unités doivent exécuter le même microprogramme d’appareil (instruction de [mise à jour du microprogramme](update-device-firmware.md)).
- Tous les appareils subordonnés doivent être démarrés avant l’appareil maître.
- La même valeur d’exposition doit être définie sur tous les appareils.
- Le paramètre de *retard par rapport au maître* de chaque appareil subordonné est relatif à l’appareil maître.

## <a name="record-when-each-unit-has-a-host-pc"></a>Enregistrer lorsque chaque unité a un PC hôte

Dans l’exemple ci-dessous, chaque appareil a son propre PC hôte dédié.
Il est recommandé de connecter les appareils à des PC dédiés afin d’éviter les problèmes liés à l’utilisation de la bande passante USB et de l’UC/GPU.

### <a name="subordinate-1"></a>Subordonné-1

1. Configurez un enregistreur pour la première unité

      `k4arecorder.exe --external-sync sub -e -8 -r 5 -l 10 sub1.mkv`

2. L’appareil commence à attendre

    ```console
    Device serial number: 000011590212
    Device version: Rel; C: 1.5.78; D: 1.5.60[6109.6109]; A: 1.5.13
    Device started
    [subordinate mode] Waiting for signal from master
    ```

### <a name="subordinate-2"></a>Subordonné-2

1. Configurez un enregistreur pour la deuxième unité

    `k4arecorder.exe --external-sync sub -e -8 -r 5 -l 10 sub2.mkv`

2. L’appareil commence à attendre

    ```console
    Device serial number: 000011590212
    Device version: Rel; C: 1.5.78; D: 1.5.60[6109.6109]; A: 1.5.13
    Device started
    [subordinate mode] Waiting for signal from master
    ```

### <a name="master"></a>Master

1. Démarrez l’enregistrement sur le maître

    `>k4arecorder.exe --external-sync master -e -8 -r 5 -l 10 master.mkv`

2. Attendez la fin de l’enregistrement

## <a name="recording-when-multiple-units-connected-to-single-host-pc"></a>Enregistrement lorsque plusieurs unités sont connectées à un seul PC hôte

Plusieurs Azure Kinect DK peuvent être connectés à un seul PC hôte. Toutefois, cela peut être très exigeant pour la bande passante USB et le calcul de l’hôte. Pour réduire la demande :

- Connectez chaque appareil à son propre contrôleur d’hôte USB.
- Dotez-vous d’un CPU puissant capable de gérer le moteur de profondeur pour chaque appareil.
- Enregistrez uniquement les capteurs nécessaires et utilisez un taux de trames inférieur.

Commencez toujours par démarrer les appareils subordonnés avant l’appareil maître.

## <a name="subordinate-1"></a>Subordonné-1

1. Démarrez l’enregistreur sur l’appareil subordonné

    `>k4arecorder.exe --device 1 --external-sync subordinate --imu OFF -e -8 -r 5 -l 5 output-2.mkv`

2. L’appareil passe en état d’attente

## <a name="master"></a>Master

1. Démarrez l’appareil maître

    `>k4arecorder.exe --device 0 --external-sync master --imu OFF -e -8 -r 5 -l 5 output-1.mkv`

2. Attendez la fin de l’enregistrement

## <a name="playing-recording"></a>Lecture de l’enregistrement

Vous pouvez utiliser la [Visionneuse Azure Kinect](azure-kinect-viewer.md) pour lire un enregistrement.



## <a name="tips"></a>Conseils

- Utilisez l’exposition manuelle pour l’enregistrement de caméras synchronisées. L’exposition automatique de la caméra RVB peut avoir une incidence sur la synchronisation.
- Un redémarrage de l’appareil subordonné entraîne une perte de la synchronisation.
- Certains [modes de caméra](hardware-specification.md#depth-camera-supported-operating-modes) prennent en charge au maximum 15 i/s. Nous vous recommandons de ne pas mélanger les modes/fréquences d’images entre les appareils.
- La connexion de plusieurs unités à un seul PC peut facilement saturer la bande passante USB. Envisagez d’utiliser un PC hôte distinct par appareil. Faites également attention au calcul de l’UC/GPU.
- Désactivez le microphone et les IMU s’ils ne sont pas nécessaires pour améliorer la fiabilité.

Pour tout problème, consultez [Résolution des problèmes](troubleshooting.md)

## <a name="see-also"></a>Voir aussi

- [Configurer une synchronisation externe](https://support.microsoft.com/help/4494429/sync-multiple-devices)
- [Enregistreur Azure Kinect](azure-kinect-recorder.md) pour les paramètres de l’enregistreur et des informations supplémentaires.
- [Visionneuse Azure Kinect](azure-kinect-viewer.md) pour la lecture des enregistrements ou la définition des propriétés de caméra RVB non disponible via l’enregistreur.
- [Outil du microprogramme Kinect Azure](azure-kinect-firmware-tool.md) pour la mise à jour du microprogramme de l’appareil.
