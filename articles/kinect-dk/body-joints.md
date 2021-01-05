---
title: Jointures de suivi du corps Azure Kinect
description: Comprenez l’image, les jointures, les coordonnées de jointures et la hiérarchie des jointures du corps dans Azure Kinect DK.
author: qm13
ms.author: quentinm
ms.reviewer: cedmonds, abalan
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, portage, corps, suivi, jointure, hiérarchie, os, connexion
ms.openlocfilehash: 4cf6ac13a93d0674f9fa144abcc3153a2d7c3350
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276512"
---
# <a name="azure-kinect-body-tracking-joints"></a>Jointures de suivi du corps Azure Kinect

Le suivi du corps Azure Kinect peut effectuer le suivi de plusieurs corps humains en même temps. Chaque corps comprend un ID pour la corrélation temporelle entre les images et le squelette cinématique. Le nombre de corps détectés dans chaque image peut être obtenu à l’aide de `k4abt_frame_get_num_bodies()`.

## <a name="joints"></a>Jointures

La position et l’orientation de la jointure sont des estimations relatives à l’image de référence du capteur de profondeur global. La position est spécifiée en millimètres. L’orientation est exprimée sous la forme d’un quaternion normalisé.

## <a name="joint-coordinates"></a>Coordonnées de jointures

La position et l’orientation de chaque jointure forment le système de coordonnées spécifique de cette jointure. Tous les systèmes de coordonnées de jointure sont des systèmes absolus par rapport au système de coordonnées 3D de la caméra de profondeur.

> [!NOTE]
> Les coordonnées de jointure figurent dans l’orientation d’axe. L’orientation d’axe est largement utilisée avec les avatars commerciaux, les moteurs de jeux et les logiciels de rendu. L’utilisation de l’orientation d’axe simplifie les mouvements en miroir, par exemple, la levée des deux bras à 20 degrés.

![Coordonnées de jointures](./media/concepts/joint-coordinates.png)

## <a name="joint-hierarchy"></a>Hiérarchie des jointures

Un squelette compte 32 jointures qui sont hiérarchisées du centre du corps aux extrémités. Chaque connexion (os) lie une jointure parente à une jointure enfant. La figure illustre les emplacements et la connexion des jointures par rapport au corps humain.

![Hiérarchie des jointures](./media/concepts/joint-hierarchy.png)

Le tableau suivant énumère les connexions de jointures standard.

|Index |Nom de jointure     | Jointure parente   |
|------|---------------|----------------|
| 0    |BASSIN         | -              |
| 1    |COLONNE_NOMBRIL    | BASSIN         |
| 2    |COLONNE_THORAX    | COLONNE_NOMBRIL    |
| 3    |COU           | COLONNE_THORAX    |
| 4    |CLAVICULE_GAUCHE  | COLONNE_THORAX    |
| 5    |ÉPAULE_GAUCHE  | CLAVICULE_GAUCHE  |
| 6    |COUDE_GAUCHE     | ÉPAULE_GAUCHE  |
| 7    |POIGNET_GAUCHE     | COUDE_GAUCHE     |
| 8    |MAIN_GAUCHE      | POIGNET_GAUCHE     |
| 9    |BOUTMAIN_GAUCHE   | MAIN_GAUCHE      |
| 10   |POUCE_GAUCHE     | POIGNET_GAUCHE     |
| 11   |CLAVICULE_DROITE | COLONNE_THORAX    |
| 12   |ÉPAULE_DROITE | CLAVICULE_DROITE |
| 13   |COUDE_DROIT    | ÉPAULE_DROITE |
| 14   |POIGNET_DROIT    | COUDE_DROIT    |
| 15   |MAIN_DROITE     | POIGNET_DROIT    |
| 16   |BOUTMAIN_DROITE  | MAIN_DROITE     |
| 17   |POUCE_DROIT    | POIGNET_DROIT    |
| 18   |HANCHE_GAUCHE       | BASSIN         |
| 19   |GENOU_GAUCHE      | HANCHE_GAUCHE       |
| 20   |CHEVILLE_GAUCHE     | GENOU_GAUCHE      |
| 21   |PIED_GAUCHE      | CHEVILLE_GAUCHE     |
| 22   |HANCHE_DROITE      | BASSIN         |
| 23   |GENOU_DROIT     | HANCHE_DROITE      |
| 24   |CHEVILLE_DROITE    | GENOU_DROIT     |
| 25   |PIED_DROIT     | CHEVILLE_DROITE    |
| 26   |HEAD           | COU           |
| 27   |NEZ           | TÊTE           |
| 28   |ŒIL_GAUCHE       | TÊTE           |
| 29   |OREILLE_GAUCHE       | TÊTE           |
| 30   |ŒIL_DROIT      | TÊTE           |
| 31   |OREILLE_DROITE      | TÊTE           |

## <a name="next-steps"></a>Étapes suivantes

[Mappage d’index de suivi de corps](body-index-map.md)
