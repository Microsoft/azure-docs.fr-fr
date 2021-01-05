---
title: Série Mv2 – Machines virtuelles Microsoft Azure
description: Spécifications pour les machines virtuelles de la série Mv2.
author: ayshakeen
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 04/07/2020
ms.author: jushiman
ms.openlocfilehash: b4de2ec68d3cd10dfc4e95c6c2232837a7fca626
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975754"
---
# <a name="mv2-series"></a>Série Mv2

La série Mv2 offre une plateforme à haut débit et faible latence sur un processeur Intel® Xeon® Platinum 8180M 2,5 GHz (Skylake) à technologie Hyper-thread avec une fréquence de base principale de 2,5 GHz et une fréquence turbo max de 3,8 GHz. Quelle que soit leur taille, toutes les machines virtuelles de la série Mv2 peuvent utiliser des disques persistants Standard et Premium. Les instances de série Mv2 sont des tailles de machine virtuelle à mémoire optimisée fournissant des performances de calcul inégalées pour prendre en charge des bases de données et des charges de travail en mémoire volumineuses, avec un ratio mémoire/processeur élevé idéal pour les serveurs de base de données relationnelle, les caches volumineux et l’analytique en mémoire.

Machines virtuelles de série Mv2 dotées de la technologie Hyper-Threading d’Intel®

[Stockage Premium](premium-storage-performance.md) : Pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Non pris en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Non pris en charge<br>
[Génération de machine virtuelle prise en charge](generation-2.md) : Générations 1 et 2<br>
[Accélérateur d’écriture](./how-to-enable-write-accelerator.md) : Pris en charge<br>
<br>

|Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS / MBps (taille du cache en Gio) | Débit du disque non mis en cache max. : IOPS / MBps | Nombre max de cartes réseau | Bande passante réseau attendue (Mbit/s) |
|---|---|---|---|---|---|---|---|---|
| Standard_M208ms_v2<sup>1</sup> | 208 | 5700 | 4096 | 64 | 80000 / 800 (7040) | 40000 / 1000 | 8 | 16000 |
| Standard_M208s_v2<sup>1</sup> | 208 | 2 850 | 4096 | 64 | 80000 / 800 (7040) | 40000 / 1000 | 8 | 16000 |
| Standard_M416ms_v2<sup>1</sup> | 416 | 11400 | 8 192 | 64 | 250000 / 1600 (14080) | 80000 / 2000 | 8 | 32000 |
| Standard_M416s_v2<sup>1</sup> | 416 | 5700 | 8 192 | 64 | 250000 / 1600 (14080) | 80000 / 2000 | 8 | 32000 |

<sup>1</sup> Les machines virtuelles de la série Mv2 sont uniquement de 2e génération et prennent en charge un sous-ensemble d’images prises en charge de 2e génération. Pour obtenir la liste complète des images prises en charge pour la série Mv2, voir ci-dessous. Si vous utilisez Linux, consultez [Support des machines virtuelles de 2e génération dans Azure](./generation-2.md) pour obtenir des instructions sur la façon de rechercher et sélectionner une image. Si vous utilisez Windows, consultez [Support des machines virtuelles de 2e génération dans Azure](./generation-2.md) pour obtenir des instructions sur la façon de rechercher et sélectionner une image. 

- Windows Server 2019 ou version ultérieure
- SUSE Linux Enterprise Server 12 SP4 et versions ultérieures ou SUSE Linux Enterprise Server 15 SP1 et versions ultérieures
- Red Hat Enterprise Linux 7.6, 7.7, 8.1 ou version ultérieure 
- Oracle Enterprise Linux 7.7 ou version ultérieure



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