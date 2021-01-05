---
title: Problèmes connus et résolution des problèmes liés à Azure Kinect
description: Découvrez quelques problèmes connus et des conseils de dépannage relatifs à l’utilisation du Kit de développement logiciel (SDK) de capteur Azure Kinect DK.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: dépannage, mise à jour, bogue, Kinect, commentaires, récupération, journalisation, conseils
ms.openlocfilehash: 9711968de061956a945fca183444dd6ebde4ca9c
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94356380"
---
# <a name="azure-kinect-known-issues-and-troubleshooting"></a>Problèmes connus et résolution des problèmes liés à Azure Kinect

Cette page décrit des problèmes connus et des conseils de dépannage relatifs à l’utilisation du Kit de développement logiciel (SDK) de capteur Azure Kinect DK. Pour des problèmes spécifiques du matériel du produit, consultez également les [pages du support technique](./index.yml).

## <a name="known-issues"></a>Problèmes connus

- Problèmes de compatibilité avec les contrôleurs hôtes USB ASMedia (par exemple, circuit microprogrammé ASM1142) :
  - Certains boîtiers utilisant un pilote USB Microsoft peuvent se débloquer.
  - De nombreux PC ont également d’autres contrôleurs hôtes. Une modification du port USB3 peut alors être utile.

Pour d’autres problèmes liés au Kit de développement logiciel (SDK) de capteur, consultez [Problèmes GitHub](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/issues).

## <a name="collecting-logs"></a>Collecte des journaux d’activité

La journalisation pour k4a.dll est activée via des variables d’environnement. Par défaut, la journalisation est envoyée à stdout et seules des erreurs et des messages critiques sont générés. Vous pouvez modifier ces paramètres de façon à ce que la journalisation accède à un fichier. Vous pouvez également ajuster la verbosité en fonction des besoins. Vous trouverez ci-dessous un exemple, pour Windows, de l’activation de la journalisation dans un fichier nommé k4a.log, qui capture des messages d’avertissement et de niveau supérieur.

1. `set K4A_ENABLE_LOG_TO_A_FILE=k4a.log`
2. `set K4A_LOG_LEVEL=w`
3. Exécutez le scénario à partir de l’invite de commandes (par exemple, lancez la visionneuse).
4. Accédez au fichier k4a.log et partagez-le.

Pour plus d’informations, consultez l’extrait du fichier d’en-tête ci-dessous :

```console
/**
* environment variables
* K4A_ENABLE_LOG_TO_A_FILE =
*    0    - completely disable logging to a file
*    log\custom.log - log all messages to the path and file specified - must end in '.log' to
*                     be considered a valid entry
*    ** When enabled this takes precedence over the value of K4A_ENABLE_LOG_TO_STDOUT
*
* K4A_ENABLE_LOG_TO_STDOUT =
*    0    - disable logging to stdout
*    all else  - log all messages to stdout
*
* K4A_LOG_LEVEL =
*    'c'  - log all messages of level 'critical' criticality
*    'e'  - log all messages of level 'error' or higher criticality
*    'w'  - log all messages of level 'warning' or higher criticality
*    'i'  - log all messages of level 'info' or higher criticality
*    't'  - log all messages of level 'trace' or higher criticality
*    DEFAULT - log all message of level 'error' or higher criticality
*/
```

La journalisation pour le Kit de développement logiciel (SDK) de suivi de corps K4ABT.dll est similaire, à ceci près que les utilisateurs doivent modifier un autre ensemble de noms de variables d’environnement :

```console
/**
* environment variables
* K4ABT_ENABLE_LOG_TO_A_FILE =
*    0    - completely disable logging to a file
*    log\custom.log - log all messages to the path and file specified - must end in '.log' to
*                     be considered a valid entry
*    ** When enabled this takes precedence over the value of K4A_ENABLE_LOG_TO_STDOUT
*
* K4ABT_ENABLE_LOG_TO_STDOUT =
*    0    - disable logging to stdout
*    all else  - log all messages to stdout
*
* K4ABT_LOG_LEVEL =
*    'c'  - log all messages of level 'critical' criticality
*    'e'  - log all messages of level 'error' or higher criticality
*    'w'  - log all messages of level 'warning' or higher criticality
*    'i'  - log all messages of level 'info' or higher criticality
*    't'  - log all messages of level 'trace' or higher criticality
*    DEFAULT - log all message of level 'error' or higher criticality
*/
```

## <a name="device-doesnt-enumerate-in-device-manager"></a>La gestionnaire de périphériques n’énumère pas l’appareil

- Vérifiez le témoin d’état à l’arrière de l’appareil. S’il apparaît orange clignotant, cela signifie que vous avez un problème de connexion USB et que l’alimentation des insuffisante. Le câble d’alimentation doit être branché au transformateur fourni. Bien que le câble d’alimentation soit muni d’un connecteur USB de type A, l’appareil nécessite plus de puissance que ne peut en fournir un port USB de PC. Ne le connectez donc pas à un port de PC ou à un concentrateur USB.
- Vérifiez que le câble d’alimentation est connecté et que vous utilisez le port USB3 pour les données.
- Essayez de modifier le port USB3 pour la connexion de données (il est recommandé d’utiliser un port USB proche de la carte mère, par exemple, à l’arrière du PC).
- Vérifiez votre câble. Un câble endommagé ou de qualité médiocre peut entraîner une défaillance d’énumération (l’appareil continue de clignoter dans le gestionnaire de périphériques).
- Si vous êtes connecté à un ordinateur portable fonctionnant sur batterie, il se peut que celui-ci limite l’alimentation du port.
- Redémarrez le PC hôte.
- Si le problème persiste, il y a peut-être un problème de compatibilité.
- Si une défaillance s’est produite pendant la mise à jour du microprogramme et que l’appareil n’a pas récupéré de lui-même, effectuez une [réinitialisation aux paramètres d’usine](https://support.microsoft.com/help/4494277/reset-azure-kinect-dk).

## <a name="azure-kinect-viewer-fails-to-open"></a>La visionneuse Azure Kinect ne s’ouvre pas

- Commencez par vérifier que le Gestionnaire de périphériques Windows énumère votre appareil.

    ![Caméras Azure Kinect dans le Gestionnaire de périphériques Windows](./media/resources/viewer-fails.png)

- Vérifiez si vous disposez d’une autre application utilisant l’appareil (par exemple, application de caméra Windows). Une seule application à la fois peut accéder à l’appareil.
- Consultez les messages d’erreur dans le journal k4aviewer.err.
- Ouvrez l’application de caméra Windows et vérifiez que cela fonctionne.
- Redémarrez l’appareil, attendez que le témoin de diffusion en continu s’éteigne avant d’utiliser l’appareil.
- Redémarrez le PC hôte.
- Vérifiez que vous utilisez les pilotes graphiques les plus récents sur votre PC.
- Si vous utilisez votre propre build du Kit de développement logiciel (SDK), essayez d’utiliser la version officielle pour résoudre le problème.
- Si le problème persiste, [collectez les journaux](troubleshooting.md#collecting-logs) et formulez des commentaires.

## <a name="cannot-find-microphone"></a>Microphone introuvable

- Commencez par vérifier que le Gestionnaire de périphériques énumère le réseau de microphones.
- Si un appareil est énuméré et fonctionne correctement dans Windows, il se peut qu’après la mise à jour du microprogramme, Windows ait affecté un autre ID de conteneur à la caméra de profondeur.
- Vous pouvez essayer de réinitialiser celle-ci en accédant au Gestionnaire de périphériques, en cliquant avec le bouton droit sur « Réseau de microphones Azure Kinect », puis en sélectionnant « Désinstaller l’appareil ». Une fois cette opération terminée, détachez et rattachez le capteur.

    ![Réseau de microphones Azure Kinect](./media/resources/mic-not-found.png)

- Après avoir redémarré la visionneuse Azure Kinect, réessayez.

## <a name="device-firmware-update-issues"></a>Problèmes de mise à jour du microprogramme de l’appareil

- Si le numéro de version indiqué après la mise à jour est incorrect, il se peut que vous deviez redémarrer l’appareil.
- Une interruption de la mise à jour du microprogramme peut avoir dégradé son état et expliquer pourquoi elle n’est pas énumérée. Détachez et rattachez l’appareil, puis attendez 60 secondes pour voir s’il récupère.
Si ce n’est pas le cas, effectuez une [réinitialisation aux paramètres d’usine](https://support.microsoft.com/help/4494277/reset-azure-kinect-dk).

## <a name="image-quality-issues"></a>Problèmes de qualité d’image

- Démarrez la [visionneuse Kinect Azure](azure-kinect-viewer.md) et vérifiez que l’appareil est positionné à l’abri de toute interférence, que le capteur n’est pas bloqué et que l’objectif est propre.
- Essayez différents modes de fonctionnement pour déterminer si le problème se produit dans un mode spécifique.
- Pour partager des problèmes de qualité d’image avec l’équipe, vous pouvez :

1) Prendre un affichage de pause sur la [visionneuse Kinect Azure](azure-kinect-viewer.md) et faire une capture d’écran ou
2) Prendre un enregistrement effectué à l’aide de l’[enregistreur Azure Kinect](azure-kinect-recorder.md), par exemple, `k4arecorder.exe -l 5 -r 5 output.mkv`

## <a name="inconsistent-or-unexpected-device-timestamps"></a>Horodatages d’appareil incohérents ou inattendus

L’appel de la fonction ```k4a_device_set_color_control``` peut entraîner des changements temporaires de synchronisation sur l’appareil qui peut prendre quelques captures pour se stabiliser. Évitez d’appeler l’API dans la boucle de capture d’images afin d’éviter la réinitialisation du calcul de synchronisation interne à chaque nouvelle image. Au lieu de cela, appelez l’API avant le démarrage de la caméra ou uniquement lorsque vous avez besoin de modifier la valeur dans la boucle de capture d’images. Évitez en particulier d’appeler la fonction ```k4a_device_set_color_control(K4A_COLOR_CONTROL_AUTO_EXPOSURE_PRIORITY)```.

## <a name="usb3-host-controller-compatibility"></a>Compatibilité du contrôleur hôte USB3

Si le gestionnaire de périphériques n’énumère pas l’appareil, cela peut être dû au fait qu’il est connecté à un contrôleur USB3 non pris en charge. 

Pour l’appareil Azure Kinect DK sur **Windows, *les seuls contrôleurs hôtes pris en charge* sont Intel** , **Texas Instruments (TI)** et **Renesas**. Le Kit de développement logiciel (SDK) Azure Kinect sous Windows s’appuie sur un ID de conteneur unifié. Celui-ci doit s’étendre aux périphériques USB 2.0 et 3.0 afin que le Kit de développement logiciel (SDK) puisse détecter les périphériques de profondeur, de couleur et audio qui se trouvent physiquement sur le même appareil. Sous Linux, un plus grand nombre de contrôleurs hôtes peuvent être pris en charge, car cette plateforme s’appuie moins sur l’ID de conteneur et davantage sur les numéros de série des appareils. 

La question des contrôleurs hôtes USB se complique encore davantage quand plusieurs contrôleurs hôtes sont installés sur un PC. Lorsque les contrôleurs hôtes sont mélangés, l’utilisateur peut rencontrer des problèmes quand certains ports fonctionnent correctement et d’autres pas du tout. Selon la façon dont les ports sont connectés au boîtier, vous pouvez voir tous les ports frontaux qui rencontrent des problèmes avec Azure Kinect.

**Windows :** Pour déterminer le contrôleur hôte que vous avez, ouvrez le Gestionnaire de périphériques

1. Afficher -> Périphériques par type 
2. Avec l’appareil Azure Kinect connecté, sélectionnez Caméras -> Caméra Azure Kinect 4K
3. Afficher -> Périphériques par connexion

![Résolution des problèmes de port USB](./media/resources/usb-troubleshooting.png)

Pour mieux comprendre quel port USB est connecté sur votre ordinateur, répétez ces étapes pour chaque port USB lorsque vous connectez l’appareil Azure Kinect DK à différents ports USB sur le PC.

## <a name="depth-camera-auto-powers-down"></a>La caméra de profondeur s’éteint automatiquement

Le laser que la caméra profondeur utilise pour calculer les données de profondeur d’image a une durée de vie limitée. Pour optimiser la durée de vie des lasers, la caméra de profondeur détecte quand les données de profondeur ne sont pas utilisées. La caméra de profondeur s’éteint lorsque l’appareil diffuse en continu pendant plusieurs minutes sans que l’ordinateur hôte lise les données. Cela a également une incidence sur la synchronisation de plusieurs appareils où les appareils subordonnés démarrent dans un état où la caméra de profondeur diffuse en continu et où les images de profondeur sont retenues en attendant que l’appareil maître commence à synchroniser les captures. Pour éviter ce problème dans des scénarios de capture avec plusieurs appareils, assurez-vous que l’appareil maître démarre dans la minute suivant le démarrage du premier appareil subordonné. 

## <a name="using-body-tracking-sdk-with-unreal"></a>Utilisation du Kit de développement logiciel (SDK) de suivi de corps avec Unreal

Pour utiliser le Kit de développement logiciel (SDK) de suivi de corps avec Unreal, assurez-vous que vous avez ajouté `<SDK Installation Path>\tools` à la variable d’environnement `PATH` et copié `dnn_model_2_0.onnx` et `cudnn64_7.dll` sur `Program Files/Epic Games/UE_4.23/Engine/Binaries/Win64`.

## <a name="next-steps"></a>Étapes suivantes

[Autres informations de support](support.md)