---
title: Accessibilité du Lecteur multimédia Azure
description: Apprenez-en davantage sur les paramètres d’accessibilité du Lecteur multimédia Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.openlocfilehash: 2231c2969bbfce1668002ad4f5f719e0b8e13de5
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "81727606"
---
# <a name="accessibility"></a>Accessibilité #

Le Lecteur multimédia Azure fonctionne avec des fonctionnalités de lecteur d’écran, telles que le Narrateur de Windows et Apple OSX/iOS VoiceOver. Des balises différentes sont disponibles pour les boutons de l’interface utilisateur, et le lecteur d’écran est capable de lire ces balises de remplacement lorsque l’utilisateur s’en sert pour naviguer. Des configurations supplémentaires peuvent être définies au niveau du système d’exploitation.

## <a name="captions-and-subtitles"></a>Légendes et sous-titres ##

À ce stade, le Lecteur multimédia Azure ne prend actuellement en charge que les sous-titres pour les événements à la demande avec le format WebVTT, et les événements en direct utilisant la norme CEA 708. Le format TTML n’est pas pris en charge pour le moment. Les sous-titres permettent au lecteur de répondre aux besoins d’un plus large public, notamment les personnes souffrant d’un handicap d’audition ou souhaitant lire simultanément dans une autre langue. Ils accroissent également l’engagement de l’utilisateur envers la vidéo, améliorent la compréhension et rendent possible l’affichage de la vidéo dans des environnements sonores particuliers, comme les lieux de travail.  

## <a name="high-contrast-mode"></a>Mode de contraste élevé ##

L’interface utilisateur par défaut du Lecteur multimédia Azure est conforme à la plupart des affichages à contraste élevé des appareils/navigateurs. Les configurations peuvent être effectuées au niveau du système d’exploitation.

## <a name="mobility-options"></a>Options de mobilité ##

### <a name="tabbing-focus"></a>Focus sur tabulation ###

Le focus sur tabulation, fourni par les normes HTML générales, est disponible dans le Lecteur multimédia Azure. Pour activer le focus sur tabulation, vous devez ajouter `tabindex=0` (ou une autre valeur si vous comprenez comment est attribué l’ordre de tabulation en HTML) à l’élément `<video>` HTML de la façon suivante : `<video ... tabindex=0>...</video>`. Sur certaines plateformes, le focus des contrôles ne peut être présent que si les contrôles sont visibles et que la plateforme prend en charge ces possibilités.

Une fois le focus sur tabulation activé, l’utilisateur final peut parcourir le lecteur vidéo et le contrôler efficacement sans faire appel à la souris. Vous pouvez accéder à chaque menu contextuel ou élément contrôlable en appuyant sur la touche Tab, et sélectionner au moyen de la touche Entrée ou de la barre d’espace. Dans un menu contextuel, le fait d’appuyer sur la touche Entrée ou sur la barre d’espace développe ce menu pour que l’utilisateur final puisse continuer à utiliser la touche Tab et sélectionner un élément de menu. Une fois que vous avez obtenu le contexte de l’élément que vous souhaitez sélectionner, appuyez de nouveau sur Entrée ou sur la barre d’espace pour terminer la sélection.

### <a name="hotkeys"></a>Touches d’accès rapide ###

Le Lecteur multimédia Azure prend en charge le contrôle par l’intermédiaire des touches d’accès rapide du clavier. Dans un navigateur web, le seul moyen de contrôler l’élément vidéo sous-jacent est de placer le focus sur le Lecteur. Une fois le focus sur le Lecteur, une touche d’accès rapide peut contrôler la fonctionnalité du Lecteur.  Le tableau ci-dessous détaille les différentes touches d’accès rapide et leur comportement associé :

| Touche d’accès rapide              | Comportement                                                                |
|----------------------|-------------------------------------------------------------------------|
| F/f                  | Le lecteur entre/quitte le mode plein écran                                  |
| M/m                  | Le volume du lecteur est désactivé/activé                                          |
| Flèches haut et bas    | Le volume du lecteur est augmenté/diminué                                    |
| Flèches gauche et droite | La progression de la vidéo augmente/diminue                                  |
| 0,1,2,3,4,5,6,7,8,9  | La progression de la vidéo passe à 0 %\-90 % en fonction de la touche enfoncée |
| Action de cliquer         | Lecture/pause de la vidéo                                                   |

## <a name="next-steps"></a>Étapes suivantes

<!---Some context for the following links goes here--->
- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)