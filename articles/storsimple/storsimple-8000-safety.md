---
title: Installer et exploiter un appareil de la gamme StorSimple 8000 en toute sécurité
description: Décrit les conventions, les instructions et les considérations de sécurité, et explique comment installer et utiliser en toute sécurité votre appareil StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/04/2017
ms.author: alkohli
ms.openlocfilehash: 3adbd292d42fd6fd2a0e149e103c17968a6f084b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94950752"
---
# <a name="safely-install-and-operate-your-storsimple-device"></a>Installer et utiliser en toute sécurité votre appareil StorSimple
![Icône Avertissement](./media/storsimple-safety/IC740879.png)
![Icône Lire l’avertissement de sécurité](./media/storsimple-safety/IC740885.png) **LIRE LES INFORMATIONS CONCERNANT LA SANTÉ ET LA SÉCURITÉ**

Prenez connaissance de toutes les informations sur la santé et la sécurité relatives à votre appareil Microsoft Azure StorSimple présentées dans cet article. Conservez tous les guides imprimés fournis avec votre appareil StorSimple pour pouvoir vous y reporter ultérieurement. Le non-respect des instructions et une configuration, une utilisation et un entretien inappropriés de ce produit peuvent augmenter le risque de blessure grave ou de mort, ou de dommages à l’appareil ou à d’autres appareils. Une [version téléchargeable de ce guide](https://www.microsoft.com/download/details.aspx?id=44233) est également disponible.

## <a name="safety-icon-conventions"></a>Conventions des icônes de sécurité
Voici les icônes utilisées dans les précautions de sécurité à observer lors de la configuration et de l’exécution de votre appareil Microsoft Azure StorSimple.

| Icône | Description |
|:--- |:--- |
| ![Icône danger](./media/storsimple-safety/IC740879.png) **DANGER !** |Signale une situation dangereuse qui, si elle n’est pas évitée, entraînera la mort ou des blessures graves. Ce terme d’avertissement est réservé aux situations extrêmement dangereuses. |
| ![Icône Avertissement](./media/storsimple-safety/IC740879.png) **AVERTISSEMENT !** |Signale une situation dangereuse qui, si elle n’est pas évitée, risque d’entraîner la mort ou des blessures graves. |
| ![Icône Avertissement](./media/storsimple-safety/IC740879.png) **ATTENTION !** |Signale une situation dangereuse qui, si elle n’est pas évitée, risque d’entraîner des blessures légères ou moyennement graves. |
| ![Icône Avis](./media/storsimple-safety/IC740881.png) **AVIS :** |Signale des informations considérées comme importantes, mais non associées à un danger. |
| ![Icône Risque d’électrocution](./media/storsimple-safety/IC740882.png) **Risque d’électrocution** |Haute tension |
| ![Icône Objet lourd](./media/storsimple-safety/IC740883.png) **Objet lourd** | |
| ![Icône Aucune pièce réparable par l’utilisateur](./media/storsimple-safety/IC740879.png) **Aucune pièce réparable par l’utilisateur** |Accès réservé au personnel formé à cet effet. |
| ![Icône Lire l’avis de sécurité](./media/storsimple-safety/IC740885.png)**Lire toutes les instructions avant de commencer** | |
| ![Icône Risque de renversement](./media/storsimple-safety/IC740886.png) **Risque de renversement** | |

## <a name="handling-precautions"></a>Précautions relatives à la manipulation
![Icône Avertissement](./media/storsimple-safety/IC740879.png) ![Icône Objet lourd](./media/storsimple-safety/IC740883.png) **AVERTISSEMENT !** 

Pour réduire le risque de blessure :

* Un boîtier complet peut peser jusqu’à 32 kg ; n’essayez pas de le soulever seul.
* Avant de déplacer le boîtier, assurez-vous toujours que deux personnes sont présentes pour en supporter la charge. Si une personne tente de soulever seule un objet de ce poids, elle risque de se blesser.
* Ne soulevez pas le boîtier à l’aide des poignées des modules d’alimentation et de refroidissement (PCM) situées à l’arrière de l’unité. Elles ne sont pas conçues pour supporter le poids du boîtier.

## <a name="connection-precautions"></a>Précautions relatives au branchement
![Icône Avertissement](./media/storsimple-safety/IC740879.png) ![Icône Risque d’électrocution](./media/storsimple-safety/IC740882.png) **AVERTISSEMENT !**

Pour réduire le risque de blessure, d’électrocution ou de mort :

* Lorsque plusieurs sources d’alimentation secteur sont utilisées, débranchez tous les câbles d’alimentation pour assurer une isolation complète.
* Débranchez l’unité de façon permanente avant de la déplacer ou si vous pensez qu’elle a été endommagée d’une quelconque façon.
* Fournissez un raccordement à la masse sûr pour les cordons d’alimentation. Vérifiez que la mise à la terre du boîtier satisfait les exigences locales et nationales avant de le mettre sous tension.
* Veillez toujours à débrancher l’alimentation avant de retirer un PCM du boîtier.
* La prise du cordon d’alimentation étant le moyen principal de débrancher l’alimentation, assurez-vous que les prises de courant sont situées à proximité du matériel et facilement accessibles.

![Icône Avertissement](./media/storsimple-safety/IC740879.png) ![Icône Risque d’électrocution](./media/storsimple-safety/IC740882.png) **AVERTISSEMENT !**

Pour réduire le risque de surchauffe ou d’incendie dû aux branchements électriques :

* Fournissez une source d’alimentation appropriée offrant une protection contre les surcharges pour répondre aux exigences décrites dans les spécifications techniques.
* Ne pas utiliser de câble d’alimentation double (« en Y »).
* Pour respecter les exigences thermiques, d’émission et de sécurité applicables, aucun couvercle ne doit être retiré et des modules enfichables ou des lecteurs vides doivent être installés dans toutes les baies.
* Assurez-vous que le matériel est utilisé d’une manière spécifiée par le fabricant. Si ce matériel est utilisé d’une manière non spécifiée par le fabricant, la protection offerte par le matériel risque d’être affectée.

![Icône Avis](./media/storsimple-safety/IC740881.png) **AVIS :**

Pour garantir le bon fonctionnement du matériel et éviter tout endommagement du produit :

* Les ports RJ45 situés à l’arrière de l’appareil sont réservés à une connexion Ethernet. Ils ne doivent en aucun cas être raccordés à un réseau de télécommunications.
* Veillez à installer l’appareil dans un rack prenant en charge une configuration de refroidissement d’avant en arrière.
* Tous les modules enfichables et les plaques vierges font partie du boîtier du système. Ils ne doivent être retirés que lorsqu’ils peuvent être remplacés immédiatement. Le système ne doit pas être exécuté si tous les modules ou plaques vierges ne sont pas en place.

## <a name="rack-system-precautions"></a>Précautions relatives au système de rack
Les exigences de sécurité suivantes sont à prendre en considération lorsque vous montez l’appareil dans une armoire à rack.

![Icône Avertissement](./media/storsimple-safety/IC740879.png) ![Icône Risque de renversement](./media/storsimple-safety/IC740886.png) **AVERTISSEMENT !**

Pour réduire le risque de blessure pouvant résulter d’un basculement :

* La conception du rack doit supporter le poids total des boîtiers installés et intégrer les dispositifs d’immobilisation appropriés pour empêcher le rack de basculer ou d’être renversé au cours de l’installation ou de l’utilisation normale.
* Pour charger un rack, remplissez-le de bas en haut. Pour le vider, procédez dans l’ordre inverse.
* N’extrayez pas plus d’un boîtier à la fois du rack afin d’éliminer tout risque de basculement du rack.

![Icône Avertissement](./media/storsimple-safety/IC740879.png) ![Icône Risque d’électrocution](./media/storsimple-safety/IC740882.png) **AVERTISSEMENT !**

Pour réduire le risque de blessure, d’électrocution ou de mort :

* Le rack doit être équipé d’un système de distribution électrique sûr. Il doit fournir une protection de surintensité pour le boîtier et ne doit pas être surchargé par le nombre total de boîtiers installés. La consommation électrique nominale indiquée sur la plaque signalétique doit être respectée.
* Le système de distribution électrique doit assurer une mise à la terre fiable de chaque boîtier du rack.
* La conception du système de distribution électrique doit tenir compte de la fuite à la terre totale des blocs d’alimentation de l’ensemble des boîtiers. Notez que chacun de ces blocs d’alimentation présente un courant de fuite à la terre de 1,0 mA maximum à 60 Hz, 264 V. Il peut être nécessaire d’afficher l’avertissement suivant sur le rack : « COURANT DE FUITE ÉLEVÉ. Connexion à la terre (masse) indispensable avant le branchement d’une alimentation. »
* Lorsque les boîtiers sont installés, le rack doit répondre aux exigences de sécurité des normes UL 60950-1 et IEC 60950-1/EN 60950-1.

![Icône Avis](./media/storsimple-safety/IC740881.png) **AVIS :**

Pour garantir le refroidissement correct du système de rack :

* Assurez-vous que la conception du rack prend en compte la température ambiante de fonctionnement maximale des boîtiers, à savoir 35 degrés Celsius. Gardez au frais la pièce dans laquelle le système de rack se trouve et vérifiez que le flux d’air provenant de la climatisation est suffisant.
* Le système fonctionne avec une installation basse pression à échappement vers l’arrière (la contre-pression créée par les portes de rack et les obstacles ne doit pas dépasser 5 pascals [0,5 mm de colonne d’eau]).

## <a name="power-cooling-module-pcm-precautions"></a>Précautions relatives aux modules d’alimentation et de refroidissement (PCM)

L’appareil est conçu pour fonctionner avec deux PCM. Chacun de ces PCM est équipé d’un bloc d’alimentation et d’un ventilateur à deux axes. En cas de situation critique, le système est capable de continuer à fonctionner normalement si l’un des blocs d’alimentation tombe en panne. Deux PCM (et donc deux blocs d’alimentation) doivent être installés en permanence. Un PCM unique ne fournit pas d’alimentation redondante. Par conséquent, la défaillance d’un PCM unique est susceptible d’entraîner des temps d’arrêt ou des pertes de données.

![Icône Avertissement](./media/storsimple-safety/IC740879.png) ![Icône Risque d’électrocution](./media/storsimple-safety/IC740882.png) **AVERTISSEMENT !**

Pour réduire le risque de blessure, d’électrocution ou de mort :

* Ne retirez pas les couvercles du PCM. Vous risqueriez de vous électrocuter en touchant l’intérieur. Pour retourner le PCM et obtenir son remplacement, [contactez le support technique Microsoft](./storsimple-8000-contact-microsoft-support.md).

![Icône Avis](./media/storsimple-safety/IC740881.png) **AVIS :**

Pour garantir le bon fonctionnement du matériel et éviter tout endommagement du produit :

* Vous devez remplacer le PCM défaillant dans un délai de 24 heures. Une fois un PCM retiré pour être remplacé, le remplacement doit être effectué dans un délai de 10 minutes.
* Ne retirez pas de PCM à moins de pouvoir installer immédiatement un PCM de rechange. Le boîtier ne doit pas être utilisé si tous les modules ne sont pas en place.

## <a name="electrostatic-discharge-esd-precautions"></a>Précautions relatives aux décharges électrostatiques

![Icône Avis](./media/storsimple-safety/IC740881.png) **AVIS :**

Respectez les précautions relatives aux décharges électrostatiques ci-après.

* Veillez à porter un bracelet antistatique approprié au poignet ou à la cheville.
* Observez toutes les précautions d’usage relatives aux décharges électrostatiques lors de la manipulation des modules et des composants.
* Évitez tout contact avec les composants de fond de panier et les connecteurs des modules.
* Les dommages causés par des décharges électrostatiques ne sont pas couverts par la garantie.

## <a name="battery-disposal-precautions"></a>Précautions relatives à la mise au rebut de la batterie

Le bloc d’alimentation utilise une batterie spéciale conçue pour protéger le contenu de la mémoire pendant les pannes électriques temporaires à court terme. Cette batterie est située dans le PCM. Gardez à l’esprit les informations suivantes concernant la batterie.

![Icône Avertissement](./media/storsimple-safety/IC740879.png) **AVERTISSEMENT !**

Pour réduire le risque de court-circuit, d’incendie, d’explosion ou de décès :

* Jetez les batteries usagées conformément aux réglementations régionales/nationales.
* Évitez de démonter, d’écraser, de chauffer au-dessus de 60 degrés Celsius ou d’incinérer la batterie. Remplacez la batterie du PCM par une batterie fournie uniquement. L’utilisation d’une autre batterie peut présenter un risque d’incendie ou d’explosion.
* Placez des embouts de protection sur les batteries si vous retirez ces dernières du bloc d’alimentation.

![Icône Avis](./media/storsimple-safety/IC740881.png) **AVIS :**

Pour l’expédition des batteries ou leur transport par voie aérienne, suivez le document de référence de l’IATA sur les batteries au lithium disponible à l’adresse [https://www.iata.org/whatwedo/cargo/dgr/Pages/lithium-batteries.aspx](https://www.iata.org/whatwedo/cargo/dgr/Pages/lithium-batteries.aspx)

Une fois que vous avez lu attentivement ces consignes de sécurité, les étapes suivantes consistent à déballer, à monter en rack et à brancher votre appareil.

## <a name="next-steps"></a>Étapes suivantes

* Pour un appareil 8100, consultez la rubrique [Installation de votre appareil StorSimple 8100](storsimple-8100-hardware-installation.md).
* Pour un appareil 8600, consultez la rubrique [Installation de votre appareil StorSimple 8600](storsimple-8600-hardware-installation.md).