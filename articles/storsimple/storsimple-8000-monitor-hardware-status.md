---
title: Composants matériels de StorSimple série 8000 et leur état | Microsoft Docs
description: Découvrez comment surveiller les composants matériels de votre appareil StorSimple via le service StorSimple Device Manager.
services: storsimple
documentationcenter: ''
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/15/2018
ms.author: alkohli
ms.openlocfilehash: 6b750659bf22e856ee8ad7368e3fea64dd7245df
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94960357"
---
# <a name="use-the-storsimple-device-manager-service-to-monitor-hardware-components-and-status"></a>Utiliser le service StorSimple Device Manager pour surveiller les composants et l’état du matériel

## <a name="overview"></a>Vue d’ensemble
Cet article décrit les différents composants physiques et logiques de votre appareil StorSimple série 8000 local. Il explique également comment surveiller l’état des composants de l’appareil à l’aide du panneau **État et intégrité du matériel** du service StorSimple Device Manager.

Le panneau **État et intégrité du matériel** affiche l’état du matériel de tous les composants de l’appareil StorSimple.

La liste des composants pour 8100 présente trois sections :

* **Composants partagés** : ces derniers ne font pas partie des contrôleurs, tels que les disques, le boîtier, les composants PCM (Power and Cooling Module) et la température PCM, la tension de ligne et les capteurs de courant de ligne.
* **Composants du contrôleur 0** – Il s’agit des composants qui résident sur le contrôleur 0, tels que le contrôleur, le connecteur et l’expander SAS, les capteurs de température de contrôleur et les différentes interfaces réseau.
* **Composants du contrôleur 1** – Il s’agit des composants qui constituent le contrôleur 1, similaires à ceux détaillés pour le contrôleur 0.

Un appareil 8600 présente des composants supplémentaires qui correspondent au boîtier EBOD (Extended Bunch of Disks). La liste des composants se divise en cinq sections. Trois de ces sections contiennent les composants du boîtier principal et sont identiques à celles décrites pour le modèle 8100. Il y a deux sections supplémentaires relatives au boîtier EBOD :

* **Composants du contrôleur 0 du boîtier EBOD** – Il s’agit des composants qui résident dans le boîtier EBOD 0, tels que le contrôleur EBOD, le connecteur et l’expander SAS, et les capteurs de température de contrôleur.
* **Composants du contrôleur 1 du boîtier EBOD** – Il s’agit des composants qui constituent le boîtier EBOD 1, similaires à ceux détaillés pour le boîtier EBOD 0.
* **Composants partagés du boîtier EBOD** – Composants présents dans les boîtiers EBOD et PCM qui ne font pas partie du contrôleur EBOD.

> [!NOTE]
> **L’état du matériel n’est pas disponible pour une appliance cloud StorSimple (8010/8020).**


## <a name="monitor-the-hardware-status"></a>Surveillance de l'état du matériel
Procédez comme suit pour afficher l'état du matériel d'un composant d’appareil :

1. Accédez à **Appareils** et sélectionnez un appareil StorSimple. Accédez à **Surveiller > Hardware health** (Intégrité matérielle).

    ![Capture d'écran montrant le panneau Paramètres du service Gestionnaire de périphériques. Sous Surveiller, l’option Intégrité matérielle est sélectionnée.](./media/storsimple-8000-monitor-hardware-status/hw-health1.png)

2. Recherchez la section **Composants matériels** et faites un choix parmi les composants disponibles. Cliquez simplement sur l’étiquette du composant pour développer la liste et afficher l’état des différents composants de l’appareil. Consultez la [liste détaillée des composants du boîtier principal](#component-list-for-primary-enclosure-of-storsimple-device) et la [liste détaillée des composants détaillées du boîtier EBOD](#component-list-for-ebod-enclosure-of-storsimple-device).

    ![Capture d’écran montrant un composant sélectionné dans la section Composants matériels. Un panneau adjacent affiche le nom et l’état des parties de ce composant.](./media/storsimple-8000-monitor-hardware-status/hw-health2.png)

3. Utilisez le codage couleur suivant pour interpréter l'état du composant :
   
   * **Coche verte** – Composant sain à l’état **OK**.
   * **Jaune** – Composant défectueux à l’état **Avertissement**.
   * **Point d’exclamation rouge** – Composant à l’état **Échec**.
   * **Blanc avec texte noir** – Composant absent.
   
   La capture d’écran suivante montre un appareil dont différents composants sont à l’état **OK**, **Avertissement** et **Échec**.
       
   ![Capture d’écran montrant trois composants dans la section Composants matériels : l’un à l’état OK, l’autre à l’état Échec et le dernier à l’état Avertissement.](./media/storsimple-8000-monitor-hardware-status/hw-health3.png)

   En développant la liste **Composants partagés**, nous pouvons voir que la mémoire NVRAM et le cluster sont défectueux.

   ![Capture d’écran montrant l’élément Composants partagés sélectionné. Dans le panneau adjacent, le NVRAM et le cluster sont à l’état Avertissement.](./media/storsimple-8000-monitor-hardware-status/hw-health5.png)

   En développant la liste **Composants du contrôleur 1**, nous pouvons voir que le nœud de cluster est en échec.  

   ![Capture d’écran montrant l’élément Composants du contrôleur 1 sélectionné. Dans le panneau adjacent, le nœud de cluster est à l’état Échec.](./media/storsimple-8000-monitor-hardware-status/hw-health4.png)  

4. Si vous rencontrez un composant dont l'état n'est pas **sain** , contactez le Support technique de Microsoft. Si les alertes sont activées sur votre appareil, vous recevrez un message d'alerte. Si vous devez remplacer un composant matériel défaillant, consultez la rubrique [Remplacement des composants matériels StorSimple](./storsimple-8000-hardware-component-replacement.md).

## <a name="component-list-for-primary-enclosure-of-storsimple-device"></a>Liste des composants du boîtier principal de l'appareil StorSimple
Le tableau suivant présente les composants physiques et logiques contenus dans le boîtier principal (sur les modèles 8100 et 8600) de votre appareil StorSimple local.

| Composant | Module | Type | Emplacement | Unité remplaçable sur site (FRU) ? | Description |
| --- | --- | --- | --- | --- | --- |
| Lecteur à l’emplacement [0-11] |Lecteurs de disque |Physique |Partagé |Oui |Les disques SSD et HDD sont représentés par une ligne dans le boîtier principal. |
| Capteur de température ambiante |Boîtier |Physique |Partagé |Non |Mesure de la température à l’intérieur du châssis. |
| Capteur de température du plan médian |Boîtier |Physique |Partagé |Non |Mesure la température du plan médian. |
| Alarme sonore |Boîtier |Physique |Partagé |Non |Indique si le sous-système d'alarme sonore du châssis est fonctionnel. |
| Boîtier |Boîtier |Physique |Partagé |Oui |Indique la présence d'un châssis. |
| Paramètres du boîtier |Boîtier |Physique |Partagé |Non |Fait référence à la façade avant du châssis. |
| Capteurs de tension de ligne |PCM |Physique |Partagé |Non |L’état de plusieurs capteurs de tension de ligne s’affiche et indique si la tension mesurée se trouve dans la plage de tolérance. |
| Capteurs de courant de ligne |PCM |Physique |Partagé |Non |L’état de plusieurs capteurs de courant de ligne s’affiche et indique si l’intensité mesurée se trouve dans la plage de tolérance. |
| Capteurs de température dans PCM |PCM |Physique |Partagé |Non |L’état de plusieurs capteurs de température tels que les capteurs Inlet et Hotspot s’affiche et indique si la température mesurée se trouve dans la plage de tolérance. |
| Alimentation électrique [0-1] |PCM |Physique |Partagé |Oui |Une ligne représente chacun des blocs d'alimentation dans les deux PCM situés à l'arrière de l’appareil. |
| Refroidissement [0-1] |PCM |Physique |Partagé |Oui |Une ligne représente chacun des quatre ventilateurs résidant dans les deux PCM. |
| Batterie [0-1] |PCM |Physique |Partagé |Oui |Une ligne représente chaque module de batterie de secours inséré dans le PCM. |
| Metis |N/A |Logical |Partagé |N/A |Affiche l'état de la batterie : si elles doivent être rechargées ou arrivent en fin de vie. |
| Cluster |N/A |Logical |Partagé |N/A |Affiche l'état du cluster créé entre les deux modules de contrôleur intégrés. |
| Nœud de cluster |N/A |Logical |Partagé |N/A |Indique l'état du contrôleur en tant que partie du cluster. |
| Quorum du cluster |N/A |Logical | |N/A |Indique la présence de l'appartenance de disque majoritaire dans le pool de stockage du disque dur. |
| Espace de données du disque dur |N/A |Logical |Partagé |N/A |Espace de stockage utilisé pour les données dans le pool de stockage de disque dur (HDD). |
| Espace de gestion du disque dur |N/A |Logical |Partagé |N/A |Espace réservé dans le pool de stockage du disque dur pour les tâches de gestion. |
| Espace du quorum du disque dur |N/A |Logical |Partagé |N/A |Espace réservé dans le pool de stockage du disque dur pour le quorum du cluster. |
| Espace de remplacement du disque dur |N/A |Logical |Partagé |N/A |Espace réservé dans le pool de stockage du disque dur pour le remplacement du contrôleur. |
| Espace de données SSD |N/A |Logical |Partagé |N/A |Espace de stockage utilisé pour les données dans le pool de stockage SSD. |
| Espace NVRAM SSD |N/A |Logical |Partagé |N/A |Espace de stockage dans le pool de stockage SSD dédié à la logique de la mémoire NVRAM. |
| Pool de stockage du disque dur |N/A |Logical |Partagé |N/A |Affiche l'état du pool de stockage logique créé à partir de disques durs de périphérique. |
| Pool de stockage SSD |N/A |Logical |Partagé |N/A |Affiche l'état du pool de stockage logique créé à partir de SSD de périphérique. |
| Contrôleur [0-1] [état] |E/S |Physique |Contrôleur |Oui |Affiche l'état du contrôleur, et s'il est en mode actif ou attente au sein du châssis. |
| Capteurs de température du contrôleur |E/S |Physique |Contrôleur |Non |L’état de plusieurs capteurs de température, tels que les capteurs du module E/S, de température du processeur, ainsi que les capteurs DIMM et PCIe, est affiché et indique si la température se situe dans la plage de tolérance. |
| Expander SAS |E/S |Physique |Contrôleur |Non |Indique l'état de l'expandeur SAS (serial attached SCSI), qui est utilisé pour connecter le stockage intégré au contrôleur. |
| Connecteur SAS [0-1] |E/S |Physique |Contrôleur |Non |Indique l'état de chaque connecteur SAS utilisé pour connecter le stockage intégré à l’expander SAS. |
| Interconnexion de plan médian SBB |E/S |Physique |Contrôleur |Non |Indique l'état du connecteur de plan médian, qui est utilisé pour connecter chaque contrôleur au plan médian. |
| Cœur de processeur |E/S |Physique |Contrôleur |Non |Indique l'état des cœurs de processeurs dans chaque contrôleur. |
| Puissance électronique du boîtier |E/S |Physique |Contrôleur |Non |Indique l'état du système d'alimentation utilisé par le boîtier. |
| Diagnostic électronique du boîtier |E/S |Physique |Contrôleur |Non |Indique l'état des sous-systèmes de diagnostic fournis par le contrôleur. |
| Baseboard Management Controller (BMC) |E/S |Physique |Contrôleur |Non |Indique l'état du BMC (baseboard management controller). Il s’agit d’un processeur de service spécialisé qui surveille le périphérique matériel via des capteurs et communique avec l'administrateur système via une connexion indépendante. |
| Ethernet |E/S |Physique |Contrôleur |Non |Indique l'état de chacune des interfaces réseau, autrement dit, des ports de gestion et de données fournis sur le contrôleur. |
| NVRAM |E/S |Physique |Contrôleur |Non |Indique l'état de la mémoire NVRAM, une mémoire vive non volatile avec batterie de secours qui sert à conserver des informations d’application critiques en cas de panne d'alimentation. |

## <a name="component-list-for-ebod-enclosure-of-storsimple-device"></a>Liste des composants du boîtier EBOD de l'appareil StorSimple
Le tableau suivant présente les composants physiques et logiques contenus dans le boîtier EBOD (uniquement sur le modèle 8600) de votre appareil StorSimple local.

| Composant | Module | Type | Emplacement | FRU ? | Description |
| --- | --- | --- | --- | --- | --- |
| Lecteur à l’emplacement [0-11] |Lecteurs de disque |Physique |Partagé |Oui |Une ligne représente chacun des disques HDD à l’avant du boîtier EBOD. |
| Capteur de température ambiante |Boîtier |Physique |Partagé |Non |Mesure de la température à l’intérieur du châssis. |
| Capteur de température du plan médian |Boîtier |Physique |Partagé |Non |Mesure la température du plan médian. |
| Alarme sonore |Boîtier |Physique |Partagé |Non |Indique si le sous-système d'alarme sonore du châssis est fonctionnel. |
| Boîtier |Boîtier |Physique |Partagé |Oui |Indique la présence d'un châssis. |
| Paramètres du boîtier |Boîtier |Physique |Partagé |Non |Fait référence à l’OPS ou à la façade avant du châssis. |
| Capteurs de tension de ligne |PCM |Physique |Partagé |Non |L’état de plusieurs capteurs de tension de ligne s’affiche et indique si la tension mesurée se trouve dans la plage de tolérance. |
| Capteurs de courant de ligne |PCM |Physique |Partagé |Non |L’état de plusieurs capteurs de courant de ligne s’affiche et indique si l’intensité mesurée se trouve dans la plage de tolérance. |
| Capteurs de température dans PCM |PCM |Physique |Partagé |Non |L’état de plusieurs capteurs de température tels que les capteurs Inlet et Hotspot s’affiche et indique si la température mesurée se trouve dans la plage de tolérance. |
| Alimentation électrique [0-1] |PCM |Physique |Partagé |Oui |Une ligne représente chacun des blocs d'alimentation dans les deux PCM situés à l'arrière de l’appareil. |
| Refroidissement [0-1] |PCM |Physique |Partagé |Oui |Une ligne représente chacun des quatre ventilateurs résidant dans les deux PCM. |
| Stockage local [HDD] |N/A |Logical |Partagé |N/A |Affiche l'état du pool de stockage logique créé à partir de disques durs de périphérique. |
| Contrôleur [0-1] [état] |E/S |Physique |Contrôleur |Oui |Affiche l'état des contrôleurs du module EBOD. |
| Capteurs de température dans EBOD |E/S |Physique |Contrôleur |Non |L’état de plusieurs capteurs de température de chaque contrôleur s’affiche et indique si la température mesurée se trouve dans la plage de tolérance. |
| Expander SAS |E/S |Physique |Contrôleur |Non |Indique l'état de l'expandeur SAS, qui est utilisé pour connecter le stockage intégré au contrôleur. |
| Connecteur SAS [0-2] |E/S |Physique |Contrôleur |Non |Indique l'état de chaque connecteur SAS utilisé pour connecter le stockage intégré à l’expander SAS. |
| Interconnexion de plan médian SBB |E/S |Physique |Contrôleur |Non |Indique l'état du connecteur de plan médian, qui est utilisé pour connecter chaque contrôleur au plan médian. |
| Puissance électronique du boîtier |E/S |Physique |Contrôleur |Non |Indique l'état du système d'alimentation utilisé par le boîtier. |
| Diagnostic électronique du boîtier |E/S |Physique |Contrôleur |Non |Indique l'état des sous-systèmes de diagnostic fournis par le contrôleur. |
| Connexion au contrôleur de périphérique |E/S |Physique |Contrôleur |Non |Indique l'état de la connexion entre le module E/S du module EBOD et le contrôleur de périphérique. |

## <a name="next-steps"></a>Étapes suivantes
* Pour utiliser le service StorSimple Device Manager pour gérer votre appareil, consultez [Utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).
* Si vous devez résoudre les problèmes d’un composant de l’appareil dont l’état est détérioré ou en échec, consultez [Indicateurs de surveillance StorSimple](storsimple-monitoring-indicators.md).
* Pour remplacer un composant matériel défectueux, consultez [Remplacement des composants matériels StorSimple](./storsimple-8000-hardware-component-replacement.md).
* Si les problèmes persistent, [contactez le support technique Microsoft](storsimple-8000-contact-microsoft-support.md).