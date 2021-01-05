---
title: Série Fsv2
description: Spécifications pour les machines virtuelles de la série Fsv2.
author: brbell
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 039045b613e57ee396201ee93c3962c5970cf02b
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91651919"
---
# <a name="fsv2-series"></a>Série Fsv2

La série Fsv2 s’exécute sur les processeurs Intel® Xeon® Platinum 8272CL (cascade Lake) et les processeurs Intel® Xeon® Platinum 8168 (Skylake). Il est doté d’une fréquence d’horloge turbo tous cœurs prolongée de 3,4 GHz et d’une fréquence turbo monocœur maximale de 3,7 GHz. Les instructions Intel® AVX-512 sont une nouveauté sur les processeurs Intel Scalable. Elles fourniront jusqu’à 2X plus de performances pour les charges de travail de traitement vectoriel pour les opérations en virgule flottante simple ou double précision. En d’autres termes, elles sont très rapides pour les charges de travail de calcul.

Machines virtuelles de la série Fsv2 dotées de la technologie Hyper-Threading d’Intel®.

[ACU](acu.md) : 195 - 210<br>
[Stockage Premium](premium-storage-performance.md) : Pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris en charge<br>
[Génération de machine virtuelle prise en charge](generation-2.md) : Générations 1 et 2<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS/Mbits/s (taille du cache en Gio) | Débit du disque non mis en cache max. : IOPS/Mbits/s | Nombre max de cartes réseau|Bande passante réseau attendue (Mbit/s) |
|---|---|---|---|---|---|---|---|---|
| Standard_F2s_v2  | 2  | 4   | 16  | 4  | 4 000/31 (32)       | 3 200/47    | 2|875   |
| Standard_F4s_v2  | 4  | 8   | 32  | 8  | 8 000/63 (64)       | 6 400/95    | 2|1750  |
| Standard_F8s_v2  | 8  | 16  | 64  | 16 | 16 000/127 (128)    | 12 800/190  | 4|3 500  |
| Standard_F16s_v2 | 16 | 32  | 128 | 32 | 32 000/255 (256)    | 25 600/380  | 4|7000  |
| Standard_F32s_v2 | 32 | 64  | 256 | 32 | 64 000/512 (512)    | 51 200/750  | 8|14000 |
| Standard_F48s_v2 | 48 | 96  | 384 | 32 | 96 000/768 (768)    | 76 800/1 100 | 8|21000 |
| Standard_F64s_v2 | 64 | 128 | 512 | 32 | 128 000/1 024 (1 024) | 80 000/1 100 | 8|28000 |
| Standard_F72s_v2<sup>1, 2</sup> | 72 | 144 | 576 | 32 | 144 000/1 152 (1 520) | 80 000/1 100 | 8|30000 |

<sup>1</sup> L’utilisation de plus de 64 processeurs virtuels nécessite l’un des systèmes d’exploitation invités pris en charge suivants :

- Windows Server 2016 ou version ultérieure
- Ubuntu 16.04 LTS or version ultérieure, avec noyau Azure adapté (noyau 4.15 ou version ultérieure)
- SLES 12 SP2 ou version ultérieure
- RHEL ou CentOS version 6.7 à 6.10, avec le package LIS fourni par Microsoft 4.3.1 (ou version ultérieure) installé
- RHEL ou CentOS version 7.3, avec le package LIS fourni par Microsoft 4.2.1 (ou version ultérieure) installé
- RHEL ou CentOS 7.6 ou version ultérieure
- Oracle Linux avec UEK4 ou version ultérieure
- Debian 9 avec le noyau backports, Debian 10 ou version ultérieure
- CoreOS avec un noyau 4.14 ou version ultérieure

<sup>2</sup> L’instance est isolée sur un matériel dédié à un client unique.

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