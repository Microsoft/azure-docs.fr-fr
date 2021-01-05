---
title: Indicateurs de surveillance de StorSimple | Microsoft Docs
description: Décrit les diodes électroluminescentes (LED) et les alarmes sonores utilisées pour analyser l’état de l’appareil StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 59dee7b9-ca6d-4fd9-96e6-a0071e8d248e
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.openlocfilehash: 97209dca7d30de037dbd21f5cc145b2941060e70
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015398"
---
# <a name="use-storsimple-monitoring-indicators-to-manage-your-device"></a>Utiliser les indicateurs de suivi StorSimple pour gérer votre appareil


## <a name="overview"></a>Vue d’ensemble
Votre appareil StorSimple comprend des diodes électroluminescentes (LED) et des alarmes que vous pouvez utiliser pour analyser l’état général de l’appareil StorSimple. Vous trouverez les indicateurs d’analyse sur les composants matériels du boîtier principal de l’appareil et sur le boîtier EBOD. Les indicateurs d’analyse peuvent être des LED ou des alarmes sonores.

Les LED peuvent prendre trois états pour indiquer l’état d’un module : vert, clignotant en alternance du vert au rouge-orange ou rouge-orange.  

* Les LED vertes correspondent à un bon état de fonctionnement.  
* Le clignotement en alternance du vert au rouge-orange indique la présence de conditions non critiques qui peuvent nécessiter l’intervention de l’utilisateur.  
* Les LED rouge-orange signalent la présence d’une erreur critique dans le module.  

Le reste de cet article décrit les différents voyants LED d’analyse, leur emplacement sur l’appareil StorSimple, ainsi que l’état de l’appareil en fonction de l’état des LED et des alarmes sonores associées éventuelles.

## <a name="front-panel-indicator-leds"></a>Voyants LED du panneau avant
Le panneau avant, également appelé *panneau de commande* indique *l’état* d’ensemble de tous les modules du système. Le panneau avant est identique sur l'appareil StorSimple principal et le boîtier EBOD, comme illustré ci-dessous.  

   ![Panneau avant de l’appareil][1]

Le panneau avant comprend les indicateurs suivants :  

1. Bouton Muet
2. Voyant LED d’alimentation (vert/rouge-orange)
3. Voyant LED de panne de module (ALLUMÉ rouge-orange/ÉTEINT)
4. Voyant LED d’erreur logique (ALLUMÉ rouge-orange/ÉTEINT)
5. Affichage de l’ID de l’unité  

La principale différence entre les LED du panneau avant de l’appareil et celles du boîtier EBOD est le **numéro d’identification de l’unité système** indiqué sur l’affichage à LED. L’ID d’unité par défaut affiché sur l’appareil est **00**, alors que l’ID d’unité par défaut affiché sur le boîtier EBOD est **01**. Cela vous permet de faire rapidement la différence entre l’appareil et le boîtier EBOD lorsque l’appareil est mis sous tension. Si votre appareil est hors tension, utilisez les informations fournies sous [Activer un nouvel appareil](storsimple-turn-device-on-or-off.md#turn-on-a-new-device) pour différencier l’appareil du boîtier EBOD.  

## <a name="front-panel-led-status"></a>État des LED du panneau avant
Utilisez le tableau suivant pour identifier l’état indiqué par les LED du panneau avant de  l’appareil ou du boîtier EBOD.  

| Alimentation du système | Panne de module | Erreur logique | Alarme | Statut |
| --- | --- | --- | --- | --- |
| Rouge-orange |OFF |OFF |N/A |Alimentation secteur coupée, fonctionnement sur l’alimentation de secours ou alimentation secteur activée mais les modules de contrôleur ont été retirés. |
| Vert |ACTIVÉ |ACTIVÉ |N/A |État de test panneau de commande à la mise sous tension (5 s) |
| Vert |OFF |OFF |N/A |Mise sous tension, tout fonctionne correctement |
| Vert |ACTIVÉ |N/A |LED de panne de PCM, LED de panne de ventilateur |Panne de PCM, panne de ventilateur, surchauffe ou température insuffisante |
| Vert |ACTIVÉ |N/A |LED de module d’E/S |Panne de module de contrôleur |
| Vert |ACTIVÉ |N/A |N/A |Erreur logique du boîtier |
| Vert |Clignote |N/A |LED d’état du module sur le module de contrôleur. LED de panne de PCM, LED de panne de ventilateur |Type de module de contrôleur installé inconnu, défaillance du bus I2C, erreur de configuration des données VPD (Vital Product Data) du module de contrôleur |

## <a name="power-cooling-module-pcm-indicator-leds"></a>Voyants LED du module d’alimentation et de refroidissement (PCM)
Vous trouverez les voyants LED du module d’alimentation et de refroidissement (PCM) à l’arrière du boîtier principal ou du boîtier EBOD de chaque module PCM. Cette rubrique explique comment utiliser les LED suivantes pour analyser l’état de votre appareil StorSimple.  

* LED de PCM du boîtier principal
* LED de PCM du boîtier EBOD

## <a name="pcm-leds-for-the-primary-enclosure"></a>LED de PCM du boîtier principal
L’appareil StorSimple est équipé d’un module PCM de 764 W, avec une batterie supplémentaire. L’illustration suivante représente le panneau de LED de l’appareil.  

   ![LED de PCM du boîtier principal][2]

Légende des LED :

1. Panne d’alimentation secteur
2. Panne de ventilateur
3. Panne de batterie
4. PCM OK
5. Panne d’alimentation CC
6. Batterie en bon état  

L’état du PCM est indiqué sur le panneau de LED. Le panneau de LED du PCM de l’appareil comprend six LED. Quatre de ces LED indiquent l’état de l’alimentation et du ventilateur. Les deux LED restantes indiquent l’état du module de batterie de secours du PCM. Vous pouvez utiliser les tableaux suivants pour déterminer l’état du PCM.  

### <a name="pcm-indicator-leds-for-power-supply-and-fan"></a>Voyants LED du PCM relatifs à l’alimentation et au ventilateur
| Statut | PCM OK (vert) | Panne d’alimentation secteur (orange) | Panne de ventilateur (orange) | Panne d’alimentation CC (orange) |
| --- | --- | --- | --- | --- |
| Absence d’alimentation secteur (vers le boîtier) |OFF |OFF |OFF |OFF |
| Absence d’alimentation secteur (ce PCM uniquement) |OFF |ACTIVÉ |OFF |ACTIVÉ |
| PCM alimenté sur secteur - OK |ACTIVÉ |OFF |OFF |OFF |
| Panne de PCM (panne de ventilateur) |OFF |OFF |ACTIVÉ |N/A |
| Panne de PCM (surampérage, surtension, surintensité) |OFF |ACTIVÉ |ACTIVÉ |ACTIVÉ |
| PCM (ventilateur hors tolérance) |ACTIVÉ |OFF |OFF |ACTIVÉ |
| Mode veille |Clignote |OFF |OFF |OFF |
| Téléchargement du microprogramme de PCM |OFF |Clignote |Clignote |Clignote |

### <a name="pcm-indicator-leds-for-the-backup-battery"></a>Voyants LED du PCM relatifs à la batterie de secours
| Statut | Batterie en bon état (vert) | Panne de batterie (orange) |
| --- | --- | --- |
| Batterie non présente |OFF |OFF |
| Batterie présente et chargée |ACTIVÉ |OFF |
| Batterie en charge ou décharge de maintenance |Clignote |OFF |
| Erreur logicielle de batterie (récupérable) |OFF |Clignote |
| Erreur matérielle de batterie (non récupérable) |OFF |ACTIVÉ |
| Batterie désamorcée |Clignote |OFF |

## <a name="pcm-leds-for-the-ebod-enclosure"></a>LED de PCM du boîtier EBOD
Le boîtier EBOD est équipé d’un module PCM de 580 W,  sans batterie supplémentaire. Le panneau PCM du boîtier EBOD comprend des voyants LED uniquement pour les alimentations et le ventilateur. L’illustration suivante représente ces LED.

   ![LED de PCM du boîtier EBOD][3] 

Vous pouvez utiliser le tableau suivant pour déterminer l’état du PCM.  

| Statut | PCM OK (vert) | Panne d’alimentation secteur (orange) | Panne de ventilateur (orange) | Panne d’alimentation CC (orange) |
| --- | --- | --- | --- | --- |
| Absence d’alimentation secteur (vers le boîtier) |OFF |OFF |OFF |OFF |
| Absence d’alimentation secteur (ce PCM uniquement) |OFF |ACTIVÉ |OFF |ACTIVÉ |
| PCM alimenté sur secteur - OK |ACTIVÉ |OFF |OFF |OFF |
| Panne de PCM (panne de ventilateur) |OFF |OFF |ACTIVÉ |X |
| Panne de PCM (surampérage, surtension, surintensité) |OFF |ACTIVÉ |ACTIVÉ |ACTIVÉ |
| PCM (ventilateur hors tolérance) |ACTIVÉ |OFF |OFF |ACTIVÉ |
| Mode veille |Clignote |OFF |OFF |OFF |
| Téléchargement du microprogramme de PCM |OFF |Clignote |Clignote |Clignote |

## <a name="controller-module-indicator-leds"></a>Voyants LED du module de contrôleur
L’appareil StorSimple comprend des LED pour le contrôleur principal et les modules de contrôleur EBOD.   

### <a name="monitoring-leds-for-the-primary-controller"></a>LED d’analyse pour le contrôleur principal
L’illustration suivante vous aide à identifier les LED du contrôleur principal. (Tous les composants sont répertoriés pour faciliter la compréhension.)  

   ![LED de surveillance - Contrôleur principal][4]

Utilisez le tableau suivant pour déterminer si le module de contrôleur fonctionne correctement.  

### <a name="controller-indicator-leds"></a>Voyants LED du contrôleur
| LED | Description |
| --- | --- |
| LED d’ID (bleue) |Indique que le module est en cours d’identification. Si la LED bleue clignote sur un contrôleur en marche, cela signifie que ce contrôleur est le contrôleur actif et que l’autre est le contrôleur de secours. Pour plus d’informations, consultez la page [Identifier un contrôleur actif sur votre appareil](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device). |
| LED de panne (orange) |Indique une panne du contrôleur. |
| LED OK (verte) |Une LED verte indique que le contrôleur est OK. Une LED verte clignotante indique une erreur de configuration de données VPD du contrôleur. |
| LED d’activité SAS (vertes) |Le vert fixe indique une connexion sans activité en cours. Le vert clignotant indique une activité continue sur la connexion. |
| LED d’état Ethernet |Le côté droit indique l’activité lien/réseau : (vert fixe) lien actif, (vert clignotant) activité réseau. Le côté gauche indique la vitesse du réseau: (jaune) 1 000 Mbits/s, (vert) 100 Mbits/s (vert) et (ÉTEINT) 10 Mbits/s. Selon le modèle de composant, ce voyant peut clignoter même si l’interface réseau n’est pas activée. |
| LED POST |Indique la progression du démarrage lorsque le contrôleur est mis sous tension. Si l’appareil StorSimple ne démarre pas, cette LED aide le support technique Microsoft à identifier à quel point du démarrage la panne s’est produite. |

> [!IMPORTANT]
> Si la LED de panne est allumée, le module de contrôleur présente un problème que vous pouvez peut-être résoudre en redémarrant le contrôleur. Contactez le support technique Microsoft si le redémarrage du contrôleur ne résout pas ce problème.  
> 
> 

### <a name="monitoring-leds-for-the-ebod-ebod-enclosure"></a>Analyse des LED de l’EBOD (boîtier EBOD)
Chaque contrôleur EBOD SAS de 6 Gbits/s possède des LED qui indiquent son état, comme illustré ci-dessous.  

  ![LED de surveillance - Boîtier EBOD][5]

Utilisez le tableau suivant pour déterminer si le module de contrôleur EBOD fonctionne correctement.  

### <a name="ebod-controller-module-indicator-leds"></a>Voyants LED du module de contrôleur EBOD
| Statut | Module d’E/S OK (vert) | Panne du module d’E/S (orange) | Activité sur les ports de l’hôte (vert) |
| --- | --- | --- | --- |
| Module de contrôleur OK |ACTIVÉ |OFF |- |
| Panne de module de contrôleur |OFF |ACTIVÉ |- |
| Aucune connexion au port hôte externe |- |- |OFF |
| Connexion au port hôte externe – aucune activité |- |- |ACTIVÉ |
| Connexion au port hôte externe - activité |- |- |Clignote |
| Erreur de métadonnées du module de contrôleur |Clignote |- |- |

## <a name="disk-drive-indicator-leds-for-the-primary-enclosure-and-ebod-enclosure"></a>Voyants LED du lecteur de disque relatifs au boîtier principal et au boîtier EBOD
L’appareil StorSimple possède des lecteurs de disque à la fois dans le boîtier principal et dans le boîtier EBOD. Chaque lecteur de disque contient des voyants LED d’analyse, comme décrit dans cette section. 

Pour les lecteurs de disque, l’état du lecteur est indiqué par une LED verte et une LED rouge-orange situées à l’avant de chaque module de support de disque. L’illustration suivante représente ces LED.

  ![LED des lecteurs de disque][6]

Utilisez le tableau suivant pour déterminer l’état de chaque lecteur de disque, qui affecte à son tour l’état d’ensemble des LED du panneau avant.  

### <a name="disk-drive-indicator-leds-for-the-ebod-enclosure"></a>Voyants LED du lecteur de disque relatifs au boîtier EBOD
| Statut | LED d’activité OK (verte) | LED de panne (rouge-orange) | LED associées du panneau de commande |
| --- | --- | --- | --- |
| Aucun disque installé |OFF |OFF |None |
| Disque installé et opérationnel |Clignotement avec l’activité |X |None |
| Identité de l’appareil SES (SCSI Enclosure Services) définie |ACTIVÉ |Clignotement 1 seconde allumé/1 seconde éteint |None |
| Erreur de bit de l’appareil SES définie |ACTIVÉ |ACTIVÉ |Erreur logique (rouge) |
| Panne du circuit d’alimentation |OFF |ACTIVÉ |Panne de module (rouge) |

## <a name="audible-alarms"></a>Alarmes sonores
Un appareil StorSimple contient des alarmes sonores associées au boîtier principal et au boîtier EBOD. Le panneau avant (également appelé panneau de commande) des deux boîtiers intègre une alarme sonore. L’alarme sonore se déclenche en présence d’une condition d’erreur/de panne. Les conditions suivantes déclenchent l’alarme :  

* Panne ou défaillance du ventilateur
* Tension hors plage
* Surchauffe ou température insuffisante
* Surcharge thermique
* Erreur système
* Erreur logique
* Panne d’alimentation
* Retrait d’un module d’alimentation et de refroidissement (PCM)  

Le tableau suivant décrit les différents états d’alarme.  

### <a name="alarm-states"></a>États d’alarme
| État d’alarme | Action | Action avec bouton muet enfoncé |
| --- | --- | --- |
| S0 |Mode normal : silencieux |Deux bips sonores |
| S1 |Mode d’erreur : 1 seconde activée/1 seconde désactivée |Transition vers S2 ou S3 (voir remarques) |
| S2 |Mode rappel : signal sonore par intermittence |None |
| S3 |Mode muet : silencieux |None |
| S4 |Mode erreur/panne critique : signal sonore continu |Non disponible : le mode muet est désactivé |

> [!NOTE]
> * Dans l’état d’alarme S1, si vous n’appuyez pas sur le bouton muet dans les 2 minutes, l’état passe automatiquement sur S2 ou S3.  
> * Les états d’alarmes S1 à S4 reviennent à S0 une fois la condition de panne supprimée.  
> * L’alarme peut passer à l’état de panne critique S4 depuis n’importe quel autre état.  


Vous pouvez désactiver l’alarme sonore en appuyant sur le bouton muet du panneau de commande. L’alarme sonore est automatiquement désactivée au bout de deux minutes si le bouton muet n’est pas actionné manuellement. Lorsque le son d’alarme est coupé, l’alarme continue d’émettre un bip bref par intermittence pour indiquer qu’un problème est toujours présent. L’alarme s’arrête une fois tous les problèmes résolus.

Le tableau suivant décrit les différentes conditions d’alarme.

### <a name="alarm-conditions"></a>Conditions d’alarme
| Statut | severity | Alarme | LED du panneau de commande |
| --- | --- | --- | --- |
| Alerte PCM : perte d’alimentation CC d’un seul PCM |Erreur : aucune perte de redondance |S1 |Panne de module |
| Alerte PCM : perte d’alimentation CC d’un seul PCM |Erreur : perte de redondance |S1 |Panne de module |
| Défaillance du ventilateur du PCM |Erreur : perte de redondance |S1 |Panne de module |
| Le module SBB a détecté une panne de PCM |Erreur |S1 |Panne de module |
| PCM retiré |Erreur de configuration |None |Panne de module |
| Erreur de configuration de boîtier |Erreur : critique |S1 |Panne de module |
| Alerte de température basse |Avertissement |S1 |Panne de module |
| Alerte de température élevée |Avertissement |S1 |Panne de module |
| Alarme de surchauffe |Erreur : critique |S1 |Panne de module |
| Défaillance du bus I2C |Erreur : perte de redondance |S1 |Panne de module |
| Erreur de communication (I2C) du panneau de commande |Erreur : critique |S1 |Panne de module |
| Erreur de contrôleur |Erreur : critique |S1 |Panne de module |
| Panne de module d’interface SBB |Erreur : critique |S1 |Panne de module |
| Panne de module d’interface SBB : aucun module opérationnel |Erreur : critique |S4 |Panne de module |
| Module d’interface SBB retiré |Avertissement |None |Panne de module |
| Erreur de contrôle d’alimentation du lecteur |Avertissement : aucune perte d’alimentation du disque |S1 |Panne de module |
| Erreur de contrôle d’alimentation du lecteur |Erreur : critique ; perte d’alimentation du lecteur |S1 |Panne de module |
| Lecture retiré |Avertissement |None |Panne de module |
| Alimentation insuffisante |Avertissement |Aucun |Panne de module |

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur [les composants matériels StorSimple et leur état](storsimple-8000-monitor-hardware-status.md).

[1]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE01.png
[2]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE02.png
[3]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE03.png
[4]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE04.png
[5]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE05.png
[6]: ./media/storsimple-monitoring-indicators/storsimple-monitoring-indicators-IMAGE06.png


