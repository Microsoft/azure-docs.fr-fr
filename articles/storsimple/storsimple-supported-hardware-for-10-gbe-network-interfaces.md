---
title: Matériel pour les interfaces StorSimple 10 GbE | Microsoft Docs
description: Décrit les transmetteurs, câbles et commutateurs à petit facteur de forme enfichables pris en charge pour les interfaces réseau 10 GbE sur votre appareil StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: df8d40c7-f5ad-4f84-93eb-779fbd5f7243
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/21/2016
ms.author: alkohli
ms.openlocfilehash: fd30c7ec15eadd3d7945349a8aae4ac824bfdc7f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954084"
---
# <a name="supported-hardware-for-the-10-gbe-network-interfaces-on-your-storsimple-device"></a>Matériel pris en charge pour les interfaces réseau 10 GbE sur votre appareil StorSimple

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble
Cet article fournit des informations sur le matériel supplémentaire fonctionnant avec votre appareil Microsoft Azure StorSimple.

## <a name="list-of-devices-tested-by-microsoft"></a>Liste des appareils testés par Microsoft
Microsoft a testé les câbles, les commutateurs et les transmetteurs enfichables infrarouges à faible encombrement suivants, pour vérifier qu'ils fonctionnent de façon optimale avec les appareils. (Les tableaux suivants sont actualisés lorsque du nouveau matériel est testé.)

### <a name="sfp-transceivers"></a>Transmetteurs SFP+
| Marque | Modèle |
| --- | --- |
| Cisco |SFP-10G-SR |

### <a name="cables"></a>Câbles
| S. Non. | Marque | Modèle |
| --- | --- | --- |
| 1. |Cisco |SFP-H10GB-CU1M |
| 2. |Cisco |SFP-H10GB-CU2M |
| 3. |Cisco |SFP-H10GB-CU3M |
| 4. |Tripp-Lite |N820 - 05M (OM3) |

### <a name="switches"></a>Commutateurs
| S. Non. | Marque | Modèle |
| --- | --- | --- |
| 1. |Cisco |N3K C3172PQ-10-GE |
| 2. |Cisco |N3K-C3048-ZM-F |
| 3. |Cisco |N5K-C5596UP-FA |

## <a name="list-of-devices-tested-in-the-field"></a>Liste des appareils testés sur site
Cette section contient la liste des appareils qui ont été déployés sur site par les clients de StorSimple. Ils n'ont pas été testés par Microsoft, mais sont susceptibles de fonctionner avec votre appareil StorSimple.

| Paramètre | Valeur |
| --- | --- |
| Marque du commutateur |Juniper |
| Modèle du commutateur |ex4550-32F |
| Version de système d’exploitation du commutateur |JunOS 12.3R9.4 |
| Modèle de panneau |Ports intégrés (PIC 0) |
| Marque du transmetteur |Juniper |
| Modèle de transmetteur |Numéro de référence 740-021308 <br></br> Numéro de référence 740-030658 |
| Version du microprogramme du transmetteur |Rev 01 Version 0.0 (indiqué) |
| Modèle de câble |Câbles de liaison duplex LC/LC 50/125µ, OM3, LSZH |
| Modèle StorSimple |8600 |
| Version du logiciel StorSimple |6.3.9600.17491 |

## <a name="list-of-devices-tested-by-oem-provider-mellanox"></a>Liste des appareils testés par le fournisseur OEM (Mellanox)
Mellanox a testé les câbles, les commutateurs et les transmetteurs enfichables infrarouges à faible encombrement suivants, pour vérifier qu'ils fonctionnent de façon optimale avec les interfaces réseau Mellanox comme les interfaces réseau 10 GbE sur votre appareil StorSimple.

### <a name="cables-and-modules-supported-by-mellanox"></a>Câbles et modules pris en charge par Mellanox
Le tableau suivant répertorie les câbles et les modules pris en charge par Mellanox. Ils n'ont pas été testés par Microsoft, mais sont susceptibles de fonctionner avec votre appareil StorSimple.

| S. Non. | Vitesse | Modèle | Description | Marque |
| --- | --- | --- | --- | --- |
| 1. |10 GbE |CAB-SFP-SFP - 1M |câble en cuivre passif SFP+ 10 Gbit/s, 1 m |Arista |
| 2. |10 GbE |CAB-SFP-SFP-2M |câble en cuivre passif SFP+ 10 Gbit/s, 2 m |Arista |
| 3. |10 GbE |CAB-SFP-SFP-3M |câble en cuivre passif SFP+ 10 Gbit/s, 3 m |Arista |
| 4. |10 GbE |CAB-SFP-SFP-5M |câble en cuivre passif SFP+ 10 Gbit/s, 5 m |Arista |
| 5. |10 GbE |Cisco SFP-H10GBCU1M |Câble SFP+ Cisco |Cisco |
| 6. |10 GbE |Cisco SFP-H10GBCU3M |Câble SFP+ Cisco |Cisco |
| 7. |10 GbE |Cisco SFP-H10GBCU5M |Câble SFP+ Cisco |Cisco |
| 8. |10 GbE |J9281B HP X242 10G |Câble de connexion directe en cuivre SFP+ vers SFP+, 1 m |HP |
| 9. |10 GbE |455883-B21 HP BLc |10 Gb SR SFP+ Opt |HP |
| 10. |10 GbE |455886-B21 HP BLc |10 Gb LR SFP+ Opt |HP |
| 11. |10 GbE |487649-B21 HP BLc |Câble en cuivre SFP+ 0,5 m 10 GbE |HP |
| 12. |10 GbE |487652-B21 HP BLc |Câble en cuivre SFP+ 1 m 10 GbE |HP |
| 13. |10 GbE |487655-B21 HP BLc |Câble en cuivre SFP+ 3 m 10 GbE |HP |
| 14. |10 GbE |487658-B21 HP BLc |Câble en cuivre SFP+ 7 m 10 GbE |HP |
| 15. |10 GbE |537963-B21 HP BLc |Câble en cuivre SFP+ 5 m 10 GbE |HP |
| 16. |10 GbE |AP784A HP |Câble en cuivre passif série C SFP+ 3 m |HP |
| 17. |10 GbE |AP785A HP |Câble en cuivre passif série C SFP+ 5 m |HP |
| 18. |10 GbE |AP818A HP |Câble en cuivre actif série B SFP+ 1 m |HP |
| 19. |10 GbE |AP819A HP |Câble en cuivre actif série B SFP+ 3 m |HP |
| 20. |10 GbE |J9150A HP |Transmetteur X132 10G SFP+ LC SR |HP |
| 21. |10 GbE |J9151A HP |Transmetteur X132 10G SFP+ LC LR |HP |
| 22. |10 GbE |J9283B HP |Câble DAC X242 10G SFP+ SFP+ 3 m |HP |
| 23. |10 GbE |J9285B HP |Câble DAC X242 10G SFP+ SFP+ 7 m |HP |
| 24. |10 GbE |JD095B HP |Câble DAC X240 10G SFP+ SFP+ 0,65 m |HP |
| 25. |10 GbE |JD096B HP |Câble DAC X240 10G SFP+ SFP+ 1,2 m |HP |
| 26. |10 GbE |JD097B HP |Câble DAD X240 10G SFP+ SFP+ 3 m |HP |
| 27. |10 GbE |MAM1Q00A-QSA Mellanox |Adaptateur QSFP vers SFP+ |Mellanox Technologies |
| 28. |10 GbE |MC2309124-006 Mt |Câble en cuivre passif 1x SFP+ vers QSFP 10 Gbit/s 24 awg 7 m |Mellanox Technologies |
| 29. |10 GbE |MC2309124-007 Mt |Câble en cuivre passif 1x SFP+ vers QSFP 10 Gbit/s 24 awg 7 m |Mellanox Technologies |
| 30. |10 GbE |MC2309130-003 Mt |Câble en cuivre passif 1x SFP+ vers QSFP 10 Gbit/s 30 awg 3 m |Mellanox Technologies |
| 31. |10 GbE |MC2309130-00A Mt |Câble en cuivre passif 1x SFP+ vers QSFP 10 Gbit/s 30 awg 0,5 m |Mellanox Technologies |
| 32. |10 GbE |MC3309124-005 Mt |Câble en cuivre passif 1x SFP+ 10 Gbit/s 24 awg 5 m |Mellanox Technologies |
| 33. |10 GbE |MC3309124-007 Mt |Câble en cuivre passif 1x SFP+ 10 Gbit/s 24 awg 7 m |Mellanox Technologies |
| 34. |10 GbE |MC3309130-003 Mt |Câble en cuivre passif 1x SFP+ 10 Gbit/s 30 awg 3 m |Mellanox Technologies |
| 35. |10 GbE |MC3309130-00A Mt |Câble en cuivre passif 1x SFP+ 10 Gbit/s 30 awg 0,5 m |Mellanox Technologies |

### <a name="switches-supported-by-mellanox"></a>Liste des commutateurs pris en charge par Mellanox
Le tableau suivant répertorie les commutateurs pris en charge par Mellanox. Ils n'ont pas été testés par Microsoft, mais sont susceptibles de fonctionner avec votre appareil StorSimple.

| S. Non. | Vitesse | Modèle | Description | Marque |
| --- | --- | --- | --- | --- |
| 1. |10 GbE |516733-B21 |Commutateur Ethernet Blade 6120XG 10 GbE HP ProCurve |HP |
| 2. |10 GbE |538113-B21 |Module de passerelle HP 10 GbE (PTM) |HP |
| 3. |10 GbE |EN4093 |Module de commutateur évolutif IBM PureFlex System Fabric EN4093 10 Gigabit |IBM |
| 4. |1 GbE |3020 |Lame de commutateur Cisco Catalyst 3020 1 GbE |Cisco |
| 5. |1 GbE |3020X |Lame de commutateur Cisco Catalyst 3020X 1 GbE |Cisco |
| 6. |1 GbE |438030-B21 |Module de commutateur 1 GbE HP - couche GbE2c 2/3 commutateur Ethernet |HP |
| 7. |1 GbE |6120G |Lame de commutateur HP ProCurve 6120G/XG 1 GbE |HP |

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur les composants matériels StorSimple et leur état](./storsimple-8000-monitor-hardware-status.md).