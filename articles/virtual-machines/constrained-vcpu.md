---
title: Tailles de processeurs virtuels restreintes
description: Liste les tailles de machine virtuelle compatibles avec un nombre restreint de processeurs virtuels.
author: mimckitt
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 03/09/2018
ms.author: mimckitt
ms.openlocfilehash: 530ae08dabe54ed279a4055e52109aad0ef4f842
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94426151"
---
# <a name="constrained-vcpu-capable-vm-sizes"></a>Tailles de machine virtuelle compatibles avec des processeurs virtuels restreints

Certaines charges de travail de base de données, comme SQL Server ou Oracle, nécessitent une mémoire, un espace de stockage et une bande passante d’E/S élevés, mais pas forcément un grand nombre de cœurs. De nombreuses charges de travail de base de données ne nécessitent pas une utilisation intensive du processeur. Azure offre certaines tailles de machine virtuelle pour lesquelles vous pouvez limiter le nombre de processeurs virtuels par machine virtuelle afin de réduire le coût des licences logicielles, tout en conservant une mémoire, un espace de stockage et une bande passante d’E/S identiques.

Le nombre de processeurs virtuels peut être réduit à la moitié ou au quart de la taille d’origine de la machine virtuelle. Pour faciliter leur identification, ces nouvelles tailles de machine virtuelle ont un suffixe qui indique le nombre de processeurs virtuels actifs.

Par exemple, la taille actuelle de machine virtuelle Standard_GS5 dispose de 32 processeurs virtuels de 448 Go de RAM, de 64 disques (jusqu'à 256 To) et de 80 000 E/S par seconde, ou 2 Gbits/s de bande passante d’E/S. Les nouvelles tailles de machine virtuelle Standard_GS5-16 et Standard_GS5-8 sont respectivement dotées de 16 et 8 processeurs virtuels actifs, mais conservent les caractéristiques de la taille Standard_GS5 en ce qui concerne la mémoire, le stockage et la bande passante d’E/S.

Les frais de licence facturés pour SQL Server ou Oracle sont limités au nouveau nombre de processeurs virtuels et les autres produits doivent être facturés selon le nouveau nombre de processeurs virtuels. Cela entraîne une augmentation de 50 à 75 % du rapport entre les caractéristiques de machine virtuelle et les processeurs virtuels actifs (facturables). Ces nouvelles tailles de machines virtuelles permettent aux charges de travail des clients d’utiliser la même quantité de bande passante E/S, de mémoire et stockage, tout en optimisant le coût des licences logicielles. Ainsi, le coût de calcul, qui inclut les frais de licence du système d’exploitation, reste le même qu’avec la taille d’origine. Pour plus d’informations, voir [Azure VM sizes for more cost-effective database workloads](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/) (Des tailles de machine virtuelle Azure pour optimiser le coût des charges de travail de base de données).


| Nom                | Processeurs virtuels | Spécifications           |
|---------------------|------|-----------------|
| Standard_M8-2ms     | 2    | Identique à M8ms    |
| Standard_M8-4ms     | 4    | Identique à M8ms    |
| Standard_M16-4ms    | 4    | Identique à M16ms   |
| Standard_M16-8ms    | 8    | Identique à M16ms   |
| Standard_M32-8ms    | 8    | Identique à M32ms   |
| Standard_M32-16ms   | 16   | Identique à M32ms   |
| Standard_M64-32ms   | 32   | Identique à M64ms   |
| Standard_M64-16ms   | 16   | Identique à M64ms   |
| Standard_M128-64ms  | 64   | Identique à M128ms  |
| Standard_M128-32ms  | 32   | Identique à M128ms  |
| Standard_E4-2s_v3   | 2    | Identique à E4s_v3  |
| Standard_E8-4s_v3   | 4    | Identique à E8s_v3  |
| Standard_E8-2s_v3   | 2    | Identique à E8s_v3  |
| Standard_E16-8s_v3  | 8    | Identique à E16s_v3 |
| Standard_E16-4s_v3  | 4    | Identique à E16s_v3 |
| Standard_E32-16s_v3 | 16   | Identique à E32s_v3 |
| Standard_E32-8s_v3  | 8    | Identique à E32s_v3 |
| Standard_E64-32s_v3 | 32   | Identique à E64s_v3 |
| Standard_E64-16s_v3 | 16   | Identique à E64s_v3 |
| Standard_E4-2s_v4   | 2    | Identique à E4s_v4  |
| Standard_E8-4s_v4   | 4    | Identique à E8s_v4  |
| Standard_E8-2s_v4   | 2    | Identique à E8s_v4  |
| Standard_E16-8s_v4  | 8    | Identique à E16s_v4 |
| Standard_E16-4s_v4  | 4    | Identique à E16s_v4 |
| Standard_E32-16s_v4 | 16   | Identique à E32s_v4 |
| Standard_E32-8s_v4  | 8    | Identique à E32s_v4 |
| Standard_E64-32s_v4 | 32   | Identique à E64s_v4 |
| Standard_E64-16s_v4 | 16   | Identique à E64s_v4 |
| Standard_E4-2ds_v4  | 2    | Identique à E4ds_v4 |
| Standard_E8-4ds_v4  | 4    | Identique à E8ds_v4 |
| Standard_E8-2ds_v4  | 2    | Identique à E8ds_v4 |
| Standard_E16-8ds_v4 | 8    | Identique à E16ds_v4|
| Standard_E16-4ds_v4 | 4    | Identique à E16ds_v4|
| Standard_E32-16ds_v4| 16   | Identique à E32ds_v4|
| Standard_E32-8ds_v4 | 8    | Identique à E32ds_v4|
| Standard_E64-32ds_v4| 32   | Identique à E64ds_v4|
| Standard_E64-16ds_v4| 16   | Identique à E64ds_v4|
| Standard_E4-2as_v4  | 2    | Identique à E4as_v4 |
| Standard_E8-4as_v4  | 4    | Identique à E8as_v4 |
| Standard_E8-2as_v4  | 2    | Identique à E8as_v4 |
| Standard_E16-8as_v4 | 8    | Identique à E16as_v4|
| Standard_E16-4as_v4 | 4    | Identique à E16as_v4|
| Standard_E32-16as_v4| 16   | Identique à E32as_v4|
| Standard_E32-8as_v4 | 8    | Identique à E32as_v4|
| Standard_E64-32as_v4| 32   | Identique à E64as_v4|
| Standard_E64-16as_v4| 16   | Identique à E64as_v4|
| Standard_E96-48as_v4| 48   | Identique à E96as_v4|
| Standard_E96-24as_v4| 24   | Identique à E96as_v4|
| Standard_GS4-8      | 8    | Identique à GS4     |
| Standard_GS4-4      | 4    | Identique à GS4     |
| Standard_GS5-16     | 16   | Identique à GS5     |
| Standard_GS5-8      | 8    | Identique à GS5     |
| Standard_DS11-1_v2  | 1    | Identique à DS11_v2 |
| Standard_DS12-2_v2  | 2    | Identique à DS12_v2 |
| Standard_DS12-1_v2  | 1    | Identique à DS12_v2 |
| Standard_DS13-4_v2  | 4    | Identique à DS13_v2 |
| Standard_DS13-2_v2  | 2    | Identique à DS13_v2 |
| Standard_DS14-8_v2  | 8    | Identique à DS14_v2 |
| Standard_DS14-4_v2  | 4    | Identique à DS14_v2 |
| Standard_M416-208s_v2 | 208    | Identique à M416s_v2|
| Standard_M416-208ms_v2 | 208    | Identique à M416ms_v2 |

## <a name="other-sizes"></a>Autres tailles
- [Optimisé pour le calcul](./sizes-compute.md)
- [Mémoire optimisée](./sizes-memory.md)
- [Optimisé pour le stockage](./sizes-storage.md)
- [GPU](./sizes-gpu.md)
- [Calcul haute performance](./sizes-hpc.md)

## <a name="next-steps"></a>Étapes suivantes
Lisez-en davantage sur les [Unités de calcul Azure (ACU)](./acu.md) pour découvrir comment comparer les performances de calcul entre les références Azure.
