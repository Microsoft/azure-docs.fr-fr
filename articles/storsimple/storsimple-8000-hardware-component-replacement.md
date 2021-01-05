---
title: Remplacement des composants matériels de la gamme StorSimple 8000 | Microsoft Docs
description: Décrit comment remplacer de façon sécurisée les PCM, la batterie, les modules de contrôleur, les contrôleurs du boîtier, les lecteurs de disques et le châssis d’un appareil StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: troubleshooting
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/02/2017
ms.author: alkohli
ms.custom: ''
ms.openlocfilehash: 12ab5a9598cc0222f5a3e64985be2e2ea9e7e2fd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014854"
---
# <a name="replace-a-hardware-component-on-your-storsimple-8000-series-device"></a>Remplacer un composant matériel sur votre appareil StorSimple série 8000

## <a name="overview"></a>Vue d’ensemble
Les didacticiels de remplacement des composants matériels décrivent les composants matériels de votre appareil Microsoft Azure StorSimple 8000 series et les étapes nécessaires pour les retirer et les remplacer. Cet article décrit les icônes de sécurité, oriente vers les didacticiels détaillés et répertorie les composants remplaçables.

> [!IMPORTANT]
> Avant de tenter de retirer ou de remplacer un composant StorSimple, passez en revue les [conventions des icônes de sécurité](#safety-icon-conventions) et les autres [précautions de sécurité](storsimple-8000-safety.md).

### <a name="safety-icon-conventions"></a>Conventions des icônes de sécurité

Le tableau suivant décrit les icônes de sécurité utilisées dans ces didacticiels. Prêtez une attention particulière à ces icônes de sécurité quand vous effectuez les étapes nécessaires pour retirer et remplacer les composants de l’appareil.

| Icône | Texte | Informations supplémentaires |
|:--- |:--- |:--- |
| ![Icône d'avertissement](./media/storsimple-hardware-component-replacement/Warning.png) |**DANGER !** |Signale une situation dangereuse qui, si elle n’est pas évitée, entraînera la mort ou des blessures graves. Cette indication est réservée aux situations les plus extrêmes. |
| ![Icône d'avertissement](./media/storsimple-hardware-component-replacement/Warning.png) |**AVERTISSEMENT !** |Signale une situation dangereuse qui, si elle n’est pas évitée, risque d’entraîner la mort ou des blessures graves. |
| ![Icône Attention](./media/storsimple-hardware-component-replacement/Caution.png) |**ATTENTION !** |Signale une situation dangereuse qui, si elle n’est pas évitée, risque d’entraîner des blessures légères ou moyennement graves. |
| ![Icône Information](./media/storsimple-hardware-component-replacement/NoticeIcon.png) |**INFORMATION :** |Signale des informations considérées comme importantes, mais non associées à un danger. |
| ![Icône Risque d’électrocution](./media/storsimple-hardware-component-replacement/Electric.png) |**Risque d’électrocution** |Indique un voltage élevé. |
| ![Icône Poids élevé](./media/storsimple-hardware-component-replacement/Weight.png) |**Poids élevé** | |
| ![Icône Aucune pièce remplaçable par l’utilisateur](./media/storsimple-hardware-component-replacement/NoUserServiceableParts.png) |**Aucune pièce remplaçable par l’utilisateur** |Accès réservé au personnel formé à cet effet. |
| ![Icône Lire les instructions](./media/storsimple-hardware-component-replacement/ReadInstructions.png) |**Lire toutes les instructions avant de commencer** | |
| ![Icône Risque de basculement](./media/storsimple-hardware-component-replacement/TipHazard.png) |**Risque de basculement** | |

### <a name="before-you-begin"></a>Avant de commencer

Familiarisez-vous avec les informations de sécurité sur votre appareil et avec les icônes de sécurité utilisées dans ce didacticiel. Pour obtenir des informations complètes, accédez à [Installer et utiliser en toute sécurité votre appareil StorSimple](storsimple-8000-safety.md) . Consultez les [précautions de sécurité](storsimple-8000-safety.md#handling-precautions) avant de manipuler votre appareil StorSimple.

Avant d’essayer de remplacer un composant, tenez compte des informations suivantes.

![Icône Avertissement](./media/storsimple-hardware-component-replacement/Warning.png) ![Icône Risque d’électrocution](./media/storsimple-hardware-component-replacement/Electric.png) **AVERTISSEMENT !**

* Raccordez-vous à la terre correctement en utilisant une décharge  électrostatique ou un tapis antistatique lors de la manipulation des modules et des composants de votre appareil StorSimple.
* Ne touchez pas les circuits. Utilisez les poignées et les guides fournis lors de la manipulation des composants dont les circuits sont apparents.

![Icône d’avertissement](./media/storsimple-hardware-component-replacement/Warning.png) ![Icône de remarque](./media/storsimple-hardware-component-replacement/NoticeIcon.png) **REMARQUE :**

Quand vous remplacez un module, **ne laissez JAMAIS une baie vide à l’arrière du boîtier**. Procurez-vous un module de remplacement ou un module vide avant de retirer le composant qui pose problème.

## <a name="hardware-component-replacement-procedures"></a>Procédures de remplacement des composants matériels

Votre appareil StorSimple série 8000 est constitué de plusieurs modules enfichables dans les boîtiers principal et/ou EBOD. Le 8100 a un seul boîtier principal, tandis que le 8600 est un appareil à deux boîtiers : un boîtier principal et un boîtier EBOD.

Les principaux composants matériels de votre appareil sont résumés dans les tableaux suivants. Cliquez sur le lien dans la colonne **Procédure de remplacement** pour accéder au didacticiel associé.

| Components | Nombre présent | Module enfichable ? | Procédure de remplacement |
|:--- |:--- |:--- |:--- |
| Châssis |1 |Non |[Remplacer le châssis sur votre appareil StorSimple](storsimple-8000-chassis-replacement.md) |
| Contrôleurs principaux |2 |Oui |[Remplacer un module de contrôleur sur votre appareil StorSimple](storsimple-8000-controller-replacement.md) |
| PCM (Module d’alimentation et de refroidissement) 764 W |2 |Oui |[Remplacer un module d’alimentation et de refroidissement (PCM, Power and Cooling Module) sur votre appareil StorSimple](storsimple-8000-power-cooling-module-replacement.md) |
| Batterie de secours |2 |Oui |[Remplacer le module de batterie de secours sur votre appareil StorSimple](storsimple-8000-battery-replacement.md) |
| Lecteurs de disque |12 |Oui |[Remplacer un lecteur de disque sur votre appareil StorSimple](storsimple-8000-disk-drive-replacement.md) |

**Table 1** : Composants matériels dans le boîtier principal

Le boîtier principal et le boîtier EBOD diffèrent quant à leurs modules d’E-S. En outre, les PCM ont des consommations en watts différentes. Les PCM du boîtier principal consomment 764 W, tandis que ceux du boîtier EBOD consomment 580 W. Les PCM du boîtier principal contiennent également un module de batterie de secours.

| Components | Nombre présent | Module enfichable ? | Procédure de remplacement |
|:--- |:--- |:--- |:--- |
| Châssis |1 |Non |[Remplacer le châssis sur votre appareil StorSimple](storsimple-8000-chassis-replacement.md) |
| Contrôleurs EBOD |2 |Oui |[Remplacer un contrôleur EBOD sur votre appareil StorSimple](storsimple-8000-ebod-controller-replacement.md) |
| PCM (Module d’alimentation et de refroidissement) 580 W |2 |Oui |[Remplacer un module d’alimentation et de refroidissement (PCM, Power and Cooling Module) sur votre appareil StorSimple](storsimple-8000-power-cooling-module-replacement.md) |
| Lecteurs de disque |12 |Oui |[Remplacer un lecteur de disque sur votre appareil StorSimple](storsimple-8000-disk-drive-replacement.md) |

**Table 2** : Composants matériels dans le boîtier EBOD

Les modules enfichables sur l’appareil sont mis en surbrillance dans les diagrammes suivants représentant l’avant et l’arrière. Vous pouvez utiliser ces diagrammes pour déterminer l’emplacement des différents modules enfichables si un remplacement est nécessaire. Le diagramme de l’avant montre les lecteurs de disque, et les diagrammes de l’arrière du boîtier EBOD et du boîtier principal montrent les modules enfichables.

![Panneau avant de l'appareil avec les lecteurs de disque](./media/storsimple-hardware-component-replacement/IC741028.png)

**Figure 1** : Avant de l’appareil

| Étiquette | Description |
|:--- |:--- |
| 0 - 11 |Lecteurs de disques (12 au total) |

Le boîtier principal et le boîtier EBOD ont tous deux des modules de support de lecteur. Le châssis hébergeant 12 disques 3,5 pouces organisés selon un format de 3 par 4.

![Fond du panier des modules du boîtier principal de l’appareil](./media/storsimple-hardware-component-replacement/IC740994.png)

**Figure 2** : Arrière du boîtier principal

| Étiquette | Description |
|:--- |:--- |
| 1 |PCM 0 |
| 2 |PCM 1 |
| 3 |Contrôleur 0 |
| 4 |Contrôleur 1 |

![Fond de panier des modules enfichables du boîtier EBOD de l’appareil](./media/storsimple-hardware-component-replacement/IC769599.png)

**Figure 3** : Arrière du boîtier EBOD

| Étiquette | Description |
|:--- |:--- |
| 1 |PCM 0 |
| 2 |PCM 1 |
| 3 |Contrôleur 0 du boîtier EBOD |
| 4 |Contrôleur 1 du boîtier EBOD |

## <a name="field-replaceable-units"></a>Unités remplaçables sur site

Les unités remplaçables sur site suivantes sont disponibles pour votre appareil StorSimple :

* Châssis (y compris le panneau de commande intégré)
* Module d’alimentation et de refroidissement (PCM) en courant alternatif 764 W
* Module d’alimentation et de refroidissement (PCM) en courant alternatif 580 W
* Lecteur de disque dur avec module de support de lecteur
* Module de contrôleur
* Module de contrôleur du boîtier EBOD
* Module de batterie de secours
* Kit de rail de montage en rack

[Contactez le Support Microsoft](storsimple-8000-contact-microsoft-support.md) pour commander une de ces unités de remplacement.

## <a name="next-steps"></a>Étapes suivantes

Passez en revue toutes les [informations de sécurité](storsimple-8000-safety.md) avant d’essayer de remplacer un composant matériel StorSimple.
