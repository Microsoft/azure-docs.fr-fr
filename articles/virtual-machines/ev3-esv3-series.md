---
title: Séries Ev3 et Esv3
description: Spécifications pour les machines virtuelles des séries Ev3 et Esv3.
author: DavidCBerry13
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: daberry
ms.openlocfilehash: 07564955e9cfa6931e461df00ef50a237a66b9b3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500680"
---
# <a name="ev3-and-esv3-series"></a>Séries Ev3 et Esv3

Les séries Ev3 et Esv3 disposent du processeur Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake) ou Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell) dans une configuration hyperthread, ce qui leur permet d’offrir ce qui se fait de mieux pour la plupart des charges de travail à usage général et d’aligner la série Ev3 sur les machines virtuelles à usage général de la plupart des autres clouds.  La mémoire a été étendue (de 7 Gio/vCPU à 8 Gio/vCPU) et les limites de disque et de réseau ont été ajustées au niveau du cœur pour s’aligner sur la transition vers l’hyperthreading. La série Ev3 constitue la suite des tailles de machines virtuelles à mémoire haute des familles D/Dv2.

## <a name="ev3-series"></a>Série Ev3

Les instances de la série Ev3 sont basées sur des fonctionnalités des processeurs Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake) ou Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell) avec la technologie Intel Turbo Boost 2.0. Les tailles des machines virtuelles de la série ESv3 sont idéales pour les applications d’entreprise nécessitant une mémoire importante.

Le stockage sur disque de données est facturé séparément des machines virtuelles. Pour utiliser les disques de stockage Premium, utilisez des machines virtuelles au format ESv3. Les tarifs et compteurs de facturation pour les tailles ESv3 sont identiques à ceux de la série Ev3.

Machines virtuelles de série Ev3 dotées de la technologie Hyper-Threading d’Intel®.

[ACU](acu.md) : 160 - 190<br>
[Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Génération 1<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire max. : IOPS / MBps en lecture / MBps en écriture | Cartes réseau (max)/Bande passante réseau |
|---|---|---|---|---|---|---|
| Standard_E2_v3  | 2  | 16  | 50   | 4  | 3000/46/23     | 2/1 000  |
| Standard_E4_v3  | 4  | 32  | 100  | 8  | 6000/93/46     | 2/2 000  |
| Standard_E8_v3  | 8  | 64  | 200  | 16 | 12000/187/93   | 4/4 000  |
| Standard_E16_v3 | 16 | 128 | 400  | 32 | 24000/375/187  | 8/8 000  |
| Standard_E20_v3 | 20 | 160 | 500  | 32 | 30000/469/234  | 8/10000 |
| Standard_E32_v3 | 32 | 256 | 800  | 32 | 48000/750/375  | 8/16 000 |
| Standard_E48_v3 | 48 | 384 | 1200 | 32 | 96000/1000/500 | 8/24 000 |
| Standard_E64_v3 | 64 | 432 | 1 600 | 32 | 96000/1000/500 | 8/30 000 |
| Standard_E64i_v3 <sup>1,2</sup> | 64 | 432 | 1 600 | 32 | 96000/1000/500 | 8/30 000 |

<sup>1</sup> Tailles avec nombre de cœurs limité disponibles.

<sup>2</sup> L’instance est isolée sur un matériel dédié à un client unique.

## <a name="esv3-series"></a>Série Esv3

Les instances de la série Esv3 sont basées sur des fonctionnalités des processeurs Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake) ou Intel® Xeon® E5-2673 v4 2,3 GHz (Broadwell) avec la technologie Intel Turbo Boost 2.0, et utilisent un stockage Premium. Les tailles des machines virtuelles de la série ESv3 sont idéales pour les applications d’entreprise nécessitant une mémoire importante.

Machines virtuelles de série Esv3 dotées de la technologie Hyper-Threading d’Intel®.

[ACU](acu.md) : 160-190<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Générations 1 et 2<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS/Mbits/s (taille du cache en Gio) | Débit de stockage temporaire et débit de rafale mis en cache : IOPS/Mbits/s<sup>3</sup> | Débit du disque non mis en cache max. : IOPS/Mbits/s |  Débit du disque non mis en cache en rafale : IOPS/Mbits/s<sup>3</sup>| Nombre de cartes réseau/bande passante réseau attendue (Mbits/s) max. |
|---|---|---|---|---|---|---|---|---|---|
| Standard_E2s_v3                | 2  | 16  | 32  | 4  | 4 000/32 (50)       | 4 000/100    | 3 200/48    | 4 000/100 | 2/1 000 |
| Standard_E4s_v3 <sup>1</sup>   | 4  | 32  | 64  | 8  | 8 000/64 (100)      | 8 000/200    | 6 400/96    | 8 000/200 | 2/2 000 |
| Standard_E8s_v3 <sup>1</sup>   | 8  | 64  | 128 | 16 | 16 000/128 (200)    | 16 000/400   | 12 800/192  | 16 000/400 | 4/4 000 |
| Standard_E16s_v3 <sup>1</sup>  | 16 | 128 | 256 | 32 | 32 000/256 (400)    | 32 000/800   | 25 600/384  | 32 000/800 | 8/8 000 |
| Standard_E20s_v3               | 20 | 160 | 320 | 32 | 40 000/320 (400)    | 40 000/1 000  | 32 000/480  | 40 000/1 000 | 8/10 000 |
| Standard_E32s_v3 <sup>1</sup>  | 32 | 256 | 512 | 32 | 64 000/512 (800)    | 64 000/1 600  | 51 200/768  | 64 000/1 600 | 8/16 000 |
| Standard_E48s_v3 <sup>1</sup>  | 48 | 384 | 768 | 32 | 96 000/768 (1 200)   | 96 000/2 000  | 76 800/1152 | 80 000/2 000 | 8/24 000 |
| Standard_E64s_v3 <sup>1</sup>  | 64 | 432 | 864 | 32 | 128 000/1 024 (1 600) | 128 000/2 000 | 80 000/1 200 | 80 000/2 000 | 8/30 000 |
| Standard_E64is_v3 <sup>2</sup> | 64 | 432 | 864 | 32 | 128 000/1 024 (1 600) | 128 000/2 000 | 80 000/1 200 | 80 000/2 000 | 8/30 000 |

<sup>1</sup> Tailles avec nombre de cœurs limité disponibles.

<sup>2</sup> L’instance est isolée sur un matériel dédié à un client unique.

<sup>3</sup> Les machines virtuelles de la série Esv3 peuvent [augmenter via le mode rafale](./disk-bursting.md) leurs performances de disque et atteindre le maximum du mode rafale pendant au plus 30 minutes à la fois.

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