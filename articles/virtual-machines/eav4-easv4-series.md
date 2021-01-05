---
title: Séries Eav4 et Easv4
description: Spécifications pour les machines virtuelles des séries Eav4 et Easv4.
author: migerdes
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 3d1b8481aa4d4a81c95643727c1eff2a4a22da2f
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94426253"
---
# <a name="eav4-and-easv4-series"></a>Séries Eav4 et Easv4

Les machines de séries Eav4 et Easv4 utilisent le processeur EPYC<sup>TM</sup> 7452 2,35 Ghz d’AMD dans une configuration multithread avec un cache L3 jusqu’à 256 Mo, ce qui améliore les options d’exécution de la plupart des charges de travail à mémoire optimisée. Les séries Eav4 et Easv4 ont les mêmes configurations de mémoire et de disque que les séries Ev3 et Esv3.

## <a name="eav4-series"></a>Série Eav4

[ACU](acu.md) : 230 - 260<br>
[Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Génération de machine virtuelle prise en charge](generation-2.md) : Générations 1 et 2<br>
<br>

Les tailles de la série Eav4 sont basées sur le processeur AMD EPYC<sup>TM</sup> 7452 de 2,35 Ghz, qui peut atteindre une Fmax renforcée de 3,35 GHz et utiliser le stockage Premium. Les tailles des machines virtuelles de la série Eav4 sont idéales pour les applications d’entreprise nécessitant une mémoire importante. Le stockage sur disque de données est facturé séparément des machines virtuelles. Pour utiliser un SSD Premium, utilisez les tailles de la série Easv4. Les tarifs et les compteurs de facturation pour les tailles Easv4 sont identiques à celles de la série Eav3.

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire max. : IOPS / MBps en lecture / MBps en écriture | Nombre max de cartes réseau | Bande passante réseau attendue (Mbit/s) |
| -----|-----|-----|-----|-----|-----|-----|-----|
| Standard\_E2a\_v4|2|16|50|4|3000 / 46 / 23|2 | 1 000 |
| Standard\_E4a\_v4|4|32|100|8|6000 / 93 / 46|2 | 2000 |
| Standard\_E8a\_v4|8|64|200|16|12000 / 187 / 93|4 | 4000 |
| Standard\_E16a\_v4|16|128|400|32|24000 / 375 / 187|8 | 8000 |
| Standard\_E20a\_v4|20|160|500|32|30000 / 468 / 234|8 | 10000 |
| Standard\_E32a\_v4|32|256|800|32|48000 / 750 / 375|8 | 16000 |
| Standard\_E48a\_v4|48|384|1200|32|96000 / 1000 (500)|8 | 24 000 |
| Standard\_E64a\_v4|64|512|1 600|32|96000 / 1000 (500)|8 | 30000 |
| Standard\_E96a\_v4|96|672|2 400|32|96000 / 1000 (500)|8 | 30000 |

## <a name="easv4-series"></a>Série Easv4

[ACU](acu.md) : 230 - 260<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Génération de machine virtuelle prise en charge](generation-2.md) : Générations 1 et 2<br>
<br>

Les tailles de la série Easv4 sont basées sur le processeur AMD EPYC<sup>TM</sup> 7452 de 2,35 Ghz, qui peut atteindre une fréquence maximale renforcée de 3,35 GHz et utiliser un SSD Premium. Les tailles des machines virtuelles de la série Easv4 sont idéales pour les applications d’entreprise nécessitant une mémoire importante.

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS / MBps (taille du cache en Gio) | Débit du disque non mis en cache max. : IOPS / MBps | Nombre max de cartes réseau | Bande passante réseau attendue (Mbit/s) |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Standard_E2as_v4|2|16|32|4|4000 / 32 (50)|3200 / 48|2 | 1 000 |
| Standard_E4as_v4|4|32|64|8|8000 / 64 (100)|6400 / 96|2 | 2000 |
| Standard_E8as_v4|8|64|128|16|16000 / 128 (200)|12800 / 192|4 | 4000 |
| Standard_E16as_v4|16|128|256|32|32 000 / 255 (400)|25600 / 384|8 | 8000 |
| Standard_E20as_v4|20|160|320|32|40000 / 320 (500)|32000 / 480|8 | 10000 |
| Standard_E32as_v4|32|256|512|32|64 000 / 510 (800)|51200 / 768|8 | 16000 |
| Standard_E48as_v4|48|384|768|32|96000 / 1020 (1200)|76800 / 1148|8 | 24 000 |
| Standard_E64as_v4|64|512|1 024|32|128000 / 1020 (1600)|80000 / 1200|8 | 30000 |
| Standard_E96as_v4 <sup>1</sup>|96|672|1344|32|192000 / 1020 (2400)|80000 / 1200|8 | 30000 |

<sup>1</sup> [Tailles avec contraintes de cœurs disponibles](./constrained-vcpu.md).

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Autres tailles et informations

- [Usage général](sizes-general.md)
- [Mémoire optimisée](sizes-memory.md)
- [Optimisé pour le stockage](sizes-storage.md)
- [Optimisé pour le GPU](sizes-gpu.md)
- [Calcul haute performance](sizes-hpc.md)
- [Générations précédentes](sizes-previous-gen.md)

Calculatrice de prix : [Calculatrice de prix](https://azure.microsoft.com/pricing/calculator/)

Pour plus d’informations sur les types de disques : [Types de disques](./disks-types.md#ultra-disk)

## <a name="next-steps"></a>Étapes suivantes

Lisez-en davantage sur les [Unités de calcul Azure (ACU)](acu.md) pour découvrir comment comparer les performances de calcul entre les références Azure.
