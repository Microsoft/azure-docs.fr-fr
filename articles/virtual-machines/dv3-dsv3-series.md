---
title: Séries Dv3 et Dsv3
description: Spécifications pour les machines virtuelles des séries Dv3 et Dsv3.
author: joelpelley
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: jushiman
ms.openlocfilehash: 3d0735b99de3e0c68b48747d69e4720a9cbb3416
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500697"
---
# <a name="dv3-and-dsv3-series"></a>Séries Dv3 et Dsv3

Les machines virtuelles de la série Dv3 s’exécutent sur les processeurs Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake), Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell), ou Intel® Xeon® E5-2673 v3 2,4 GHz (Haswell) dans une configuration hyper-thread, ce qui offre une meilleure valeur pour la plupart des charges de travail universelles. La mémoire a été étendue (d’environ 3,5 Gio/vCPU à 4 Gio/vCPU) et les limites de disque et de réseau ont été ajustées au niveau du cœur pour s’aligner sur la transition vers l’hyperthreading. La série Dv3 n’offre plus les tailles de machine virtuelle à mémoire élevée de la série D/Dv2, qui ont été déplacées vers les séries [Ev3 et Esv3](ev3-esv3-series.md) à mémoire optimisée.

Voici des exemples de cas d’usage de la série D : applications de classe Entreprise, bases de données relationnelles, mise en cache en mémoire et analytique.

## <a name="dv3-series"></a>Série Dv3

Les tailles de la série Dv3 s'exécutent sur les processeurs Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake), Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell), ou Intel® Xeon® E5-2673 v3 2,4 GHz (Haswell) avec la technologie [Intel&reg; Turbo Boost 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html). Les tailles des machines virtuelles de la série Dv3 offrent une combinaison de processeur virtuel, mémoire et stockage temporaire pour la plupart des charges de travail de production.

Le stockage sur disque de données est facturé séparément des machines virtuelles. Pour utiliser les disques de stockage Premium, utilisez des machines virtuelles au format Dsv3. Les tarifs et compteurs de facturation pour les tailles Dsv3 sont identiques à celles de la série Dv3.

Machines virtuelles de série Dv3 dotées de la technologie Hyper-Threading d’Intel®.

[ACU](acu.md) : 160-190<br>
[Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Génération 1<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire max. : IOPS/Mbits/s en lecture/Mbits/s en écriture | Nombre de cartes réseau/bande passante réseau max. |
|---|---|---|---|---|---|---|
| Standard_D2_v3  | 2  | 8   | 50   | 4  | 3000/46/23     | 2/1 000  |
| Standard_D4_v3  | 4  | 16  | 100  | 8  | 6000/93/46     | 2/2 000  |
| Standard_D8_v3  | 8  | 32  | 200  | 16 | 12000/187/93   | 4/4 000  |
| Standard_D16_v3 | 16 | 64  | 400  | 32 | 24000/375/187  | 8/8 000  |
| Standard_D32_v3 | 32 | 128 | 800  | 32 | 48000/750/375  | 8/16 000 |
| Standard_D48_v3 | 48 | 192 | 1200 | 32 | 96000/1000/500 | 8/24 000 |
| Standard_D64_v3 | 64 | 256 | 1 600 | 32 | 96000/1000/500 | 8/30 000 |

## <a name="dsv3-series"></a>Dsv3-series

Les tailles de la série Dsv3 s'exécutent sur les processeurs Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake), Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell), ou Intel® Xeon® E5-2673 v3 2,4 GHz (Haswell) avec la technologie [Intel&reg; Turbo Boost 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html), et elles utilisent le Stockage Premium. Les tailles des machines virtuelles de la série Dsv3 offrent une combinaison de processeur virtuel, mémoire et stockage temporaire pour la plupart des charges de travail de production.

Machines virtuelles de série Dsv3 dotées de la technologie Hyper-Threading d’Intel®.

[ACU](acu.md) : 160-190<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Générations 1 et 2<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS/Mbits/s (taille du cache en Gio) | Débit de stockage temporaire et débit maximal de rafale mis en cache : IOPS/Mo/s<sup>1</sup> | Débit du disque non mis en cache max. : IOPS/Mbits/s | Débit du disque maximal de rafale non mis en cache : IOPS/Mo/s<sup>1</sup> | Nombre de cartes réseau/bande passante réseau attendue (Mbits/s) max. |
|---|---|---|---|---|---|---|---|---|---|
| Standard_D2s_v3  | 2  | 8   | 16  | 4  | 4 000/32 (50)       | 4 000/100    |3 200/48    | 4 000/100   | 2/1 000  |
| Standard_D4s_v3  | 4  | 16  | 32  | 8  | 8 000/64 (100)      | 8 000/200    |6 400/96    | 8 000/200   | 2/2 000  |
| Standard_D8s_v3  | 8  | 32  | 64  | 16 | 16 000/128 (200)    | 16 000/400   |12 800/192  | 16 000/400  | 4/4 000  |
| Standard_D16s_v3 | 16 | 64  | 128 | 32 | 32 000/256 (400)    | 32 000/800   |25 600/384  | 32 000/800  | 8/8 000  |
| Standard_D32s_v3 | 32 | 128 | 256 | 32 | 64 000/512 (800)    | 64 000/1 600  |51 200/768  | 64 000/1 600 | 8/16 000 |
| Standard_D48s_v3 | 48 | 192 | 384 | 32 | 96 000/768 (1 200)   | 96 000/2 000  |76 800/1152 | 80 000/2 000 | 8/24 000 |
| Standard_D64s_v3 | 64 | 256 | 512 | 32 | 128 000/1 024 (1 600) | 128 000/2 000 |80 000/1 200 | 80 000/2 000 | 8/30 000 |

<sup>1</sup> Les machines virtuelles de la série Dsv3 peuvent [augmenter via le mode rafale](./disk-bursting.md) leurs performances de disque et atteindre le maximum du mode rafale pendant au plus 30 minutes à la fois.

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Autres tailles et informations

- [Usage général](sizes-general.md)
- [Mémoire optimisée](sizes-memory.md)
- [Optimisé pour le stockage](sizes-storage.md)
- [Optimisé pour le GPU](sizes-gpu.md)
- [Calcul haute performance](sizes-hpc.md)
- [Générations précédentes](sizes-previous-gen.md)
- [Calculatrice de prix](https://azure.microsoft.com/pricing/calculator/)
- Pour plus d’informations sur les types de disques, consultez [Quels sont les types de disque disponibles dans Azure ?](disks-types.md)

## <a name="next-steps"></a>Étapes suivantes

Lisez-en davantage sur les [Unités de calcul Azure (ACU)](acu.md) pour découvrir comment comparer les performances de calcul entre les références Azure.