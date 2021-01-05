---
title: Recommandations sur le réseau de microphones pour le SDK Speech Devices
titleSuffix: Azure Cognitive Services
description: Recommandations sur le réseau de microphones pour le SDK Speech Devices. Ces géométries de réseau sont recommandées pour une utilisation avec la pile audio Microsoft.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/16/2019
ms.author: erhopf
ms.openlocfilehash: 698a1d52af6c2472d6c025851ead1a0b6a6dff82
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015339"
---
# <a name="speech-devices-sdk-microphone-array-recommendations"></a>Recommandations sur le réseau de microphones pour le SDK Speech Devices

Dans cet article, vous apprendrez à créer un réseau de microphones pour le SDK Speech Devices.

Le SDK Speech Devices fonctionne mieux avec un réseau de microphones conçu conformément aux directives suivantes, y compris la géométrie du microphone et la sélection des composants. Vous bénéficiez également de conseils sur l'intégration et les considérations électriques.

## <a name="microphone-geometry"></a>Géométrie du microphone

Les géométries de réseau suivantes sont recommandées pour une utilisation avec la pile audio Microsoft. La localisation des sources sonores et le rejet du bruit ambiant sont améliorés avec un plus grand nombre de microphones, selon les applications spécifiques, les scénarios utilisateur et le facteur de forme de l’appareil.

| Micros & Geometrie | Tableau circulaire | Tableau circulaire | Tableau linéaire | Tableau linéaire |
| --- | -------------- | --- | ------------ | --- |
|     | <img src="media/speech-devices-sdk/7-mic-c.png" alt="7 mic circular array" width="150"/> | <img src="media/speech-devices-sdk/4-mic-c.png" alt="4 mic circular array" width="150"/> | <img src="media/speech-devices-sdk/4-mic-l.png" alt="4 mic linear array" width="150"/> | <img src="media/speech-devices-sdk/2-mic-l.png" alt="2 mic linear array" width="150"/> |
| \# Micros | 7 | 4 | 4 | 2 |
| Géométrie | 6 à l'extérieur, 1 au centre, rayon = 42,5 mm, espacement régulier | 3 à l'extérieur, 1 au centre, rayon = 42,5 mm, espacement régulier | Longueur = 120 mm, espacement = 40 mm | Espacement = 40 mm |

Les canaux du microphone doivent être classés selon la numérotation indiquée pour chacun des tableaux ci-dessus, en partant de 0. La pile audio Microsoft nécessite un flux de référence supplémentaire de lecture audio pour annuler l'écho.

## <a name="component-selection"></a>Sélection des composants

Les composants du microphone doivent être sélectionnés pour reproduire avec précision un signal exempt de bruit et de distorsion.

Les propriétés recommandées lors de la sélection des microphones sont les suivantes :

| Paramètre | Recommandé |
| --------- | ----------- |
| SNR | \> = 65 dB (signal 1 kHz 94 dBSPL, bruit avec pondération A) |
| Mise en correspondance de l’amplitude | ± 1 dB @ 1 kHz |
| Mise en correspondance de la phase | ± 2° @ 1 kHz |
| Acoustic Overload Point (AOP) | \> = 120 dBSPL (THD = 10%) |
| Vitesse de transmission | 24 bits minimum |
| Taux d’échantillonnage | 16 kHz minimum\* |
| Réponse en fréquence | ± 3 dB, 200-8000 Hz avec masque flottant\* |
| Fiabilité | Plage de température de stockage de -40°C à 70°C<br />Plage de température de fonctionnement de -20°C à 55°C |

\*_Des taux d'échantillonnage plus élevés ou des gammes de fréquences « plus larges » peuvent être nécessaires pour des applications de communications de haute qualité (VoIP)_

Une bonne sélection des composants doit s'accompagner d'une bonne intégration électroacoustique afin de ne pas nuire aux performances des composants utilisés. Des cas d'utilisation uniques peuvent également nécessiter des exigences supplémentaires (plages de température de fonctionnement, par exemple).

## <a name="microphone-array-integration"></a>Intégration du réseau de microphones

Les performances du réseau de microphones lorsqu’il est intégré à un appareil diffèrent de la spécification du composant. Il est important de s’assurer que les microphones sont bien mis en correspondance après l’intégration. Par conséquent, les performances de l’appareil mesurées après un gain fixe ou un égaliseur doivent répondre aux recommandations suivantes :

| Paramètre          | Recommandé                                        |
| ------------------ | -------------------------------------------------- |
| SNR                | \> 63 dB (signal 1 kHz 94 dBSPL, bruit avec pondération A) |
| Sensibilité en sortie | -26 dBFS/Pa @ 1 kHz (recommandé)                  |
| Mise en correspondance de l’amplitude | ± 2 dB, 200-8000 Hz                                |
| THD%\*             | ≤ 1 %, 200-8000 Hz, 94 dBSPL, 5e ordre             |
| Réponse en fréquence | ± 6 dB, 200-8000 Hz avec masque flottant\*\*              |

\*\*_Un haut-parleur à faible distorsion est requis pour mesurer le THD (par exemple, un Neumann KH120)_

\*\*_Des gammes de fréquences « plus larges » peuvent être nécessaires pour des applications de communications de haute qualité (VoIP)_

## <a name="speaker-integration-recommendations"></a>Recommandations d’intégration du haut-parleur

Comme l’annulation de l’écho est nécessaire pour les appareils de reconnaissance vocale équipés de haut-parleurs, des recommandations supplémentaires sont fournies pour la sélection et l'intégration des haut-parleurs.

| Paramètre | Recommandé |
| --------- | ----------- |
| Considérations relatives à la linéarité | Aucun traitement non linéaire après la référence du haut-parleur. Sinon, un flux de référence de bouclage basé sur le matériel est nécessaire |
| Bouclage du haut-parleur | Fourni via WASAPI, des API privées, un plug-in ALSA personnalisé (Linux), ou via un canal de microprogramme |
| % THD | 3e bande d’octave minimum 5e ordre, lecture 70 dBA @ 0,8 m ≤ 6,3 %, 315-500 Hz ≤ 5 %, 630-5000 Hz |
| Couplage de l’écho aux microphones | \> -10 dB TCLw avec la méthode ITU-T G.122 Annex B.4, normalisée au niveau du microphone<br />TCLw = TCLwmeasured \+ (Niveau mesuré - sensibilité en sortie cible)<br />TCLw = TCLwmeasured \+ (Niveau mesuré - (-26)) |

## <a name="integration-design-architecture"></a>Intégration de l’architecture de conception

Les directives suivantes concernant l'architecture sont nécessaires lors de l'intégration de microphones à un appareil :

| Paramètre | Recommandation |
| --------- | -------------- |
| Similarité du port microphone | Tous les ports microphone ont la même longueur dans le tableau |
| Dimensions du port microphone | Taille du port : Ø0,8-1.0 mm. Longueur/diamètre du port : \< 2 |
| Scellement du microphone         | Joints d'étanchéité uniformément insérés dans l’appareil. Taux de compression recommandé \> 70 % pour les joints en mousse |
| Fiabilité du microphone     | Une grille doit être installée pour empêcher les infiltrations de poussière (entre le circuit imprimé pour les microphones à port inférieur et le joint d'étanchéité/couvercle supérieur) |
| Isolation du microphone       | Joints en caoutchouc et découplage des vibrations à travers la structure, en particulier pour isoler les trajets de vibrations dus à des haut-parleurs intégrés |
| Horloge d’échantillonnage      | L'audio de l'appareil doit être exempt de gigue et de parasites, avec une faible dérive |
| Fonctionnalité d’enregistrement   | L'appareil doit être capable d'enregistrer simultanément des flux Raw de canaux individuels |
| USB                 | Tous les périphériques d'entrée audio USB doivent définir des descripteurs conformément à la spécification [USB Audio Devices Rev3 Spec](https://www.usb.org/document-library/usb-audio-devices-rev-30-and-adopters-agreement) |
| Géométrie du microphone | Les pilotes doivent implémenter correctement les [descripteurs de géométrie du réseau de microphones](/windows-hardware/drivers/audio/ksproperty-audio-mic-array-geometry) |
| Détectabilité     | Les appareils ne doivent avoir ni matériel ou microprogramme non détectable ou non contrôlable, ni algorithmes de traitement audio non linéaires basés sur un logiciel tiers de/vers l'appareil |
| Format de capture      | Les formats de capture doivent utiliser une fréquence d’échantillonnage minimale de 16 kHz et une profondeur recommandée de 24 bits |

## <a name="electrical-architecture-considerations"></a>Considérations relatives à l’architecture électrique

Le cas échéant, les réseaux peuvent être connectés à un hôte USB (par exemple un SoC qui exécute la pile audio Microsoft) ainsi qu'à des interfaces vers des services Speech ou d'autres applications.

Les composants matériels tels que la conversion PDM à TDM doivent garantir que la plage dynamique et le rapport signal/bruit des microphones sont préservés dans les rééchantillonneurs.

La norme High-speed USB Audio Class 2.0 devrait être prise en charge dans tous les MCU audio afin de fournir la bande passante nécessaire pour un maximum de sept canaux à des fréquences d'échantillonnage et des profondeurs de bits supérieures.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur le SDK Speech Devices](speech-devices-sdk.md)