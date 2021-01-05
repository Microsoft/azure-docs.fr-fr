---
title: Vue d’ensemble de l’unité Compute Azure
description: Vue d’ensemble du concept d’unités Compute Azure. L’unité de calcul Azure offre un moyen de comparer les performances d’UC entre des références Azure.
author: mimckitt
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: mimckitt
ms.reviewer: davberg
ms.openlocfilehash: ba8c3f730897c01c52aaf4499269425426d0b7b5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91667178"
---
# <a name="azure-compute-unit-acu"></a>Unité de calcul Azure (ACU)

Le concept d’unité Compute Azure (ACU) permet de comparer les performances de calcul (UC) des références SKU Azure. Cela vous aidera à identifier facilement la référence la plus à même de répondre à vos besoins en termes de performances. L’unité ACU est actuellement normalisée sur une petite machine virtuelle (Standard_A1) correspondant à 100, et toutes les autres références représentent ensuite approximativement la rapidité avec laquelle la référence en question peut exécuter un test d’évaluation.

*Les ACU utilisent la technologie Intel® Turbo pour augmenter la fréquence du processeur et améliorer les performances.  L’augmentation des performances peut varier en fonction de la taille de la machine virtuelle, de la charge de travail et d’autres charges de travail s’exécutant sur le même hôte.

**Les ACU utilisent la technologie AMD® Boost pour augmenter la fréquence du processeur et améliorer les performances.  L’augmentation des performances peut varier en fonction de la taille de la machine virtuelle, de la charge de travail et d’autres charges de travail s’exécutant sur le même hôte.

***Hyper-Threading et capacité d’exécution de la virtualisation imbriquée

> [!IMPORTANT]
> Cette unité ACU est fournie uniquement à titre indicatif. Les résultats de votre charge de travail peuvent varier.
<br>

| Famille de références | ACU \ Processeur virtuel | vCPU : Cœur |
| --- | --- |---|
| [A0](sizes-previous-gen.md) |50 | 1:1 |
| [A1 - A4](sizes-previous-gen.md) |100 | 1:1 |
| [A5 - A7](sizes-previous-gen.md) |100 | 1:1 |
| [A1_v2 - A8_v2](sizes-general.md) |100 | 1:1 |
| [A2m_v2 - A8m_v2](sizes-general.md) |100 | 1:1 |
| [A8 - A11](sizes-previous-gen.md) |225* | 1:1 |
| [B](sizes-b-series-burstable.md) |Variable | Variable |
| [D1 - D14](sizes-previous-gen.md) |160 - 250 | 1:1 |
| [D1_v2 - D15_v2](dv2-dsv2-series.md) |210 - 250* | 1:1 |
| [DS1 - DS14](sizes-previous-gen.md) |160 - 250 | 1:1 |
| [DS1_v2 - DS15_v2](dv2-dsv2-series.md) |210 - 250* | 1:1 |
| [D_v3](dv3-dsv3-series.md) |160 - 190* | 2:1\*\*\* |
| [Ds_v3](dv3-dsv3-series.md) |160 - 190* | 2:1\*\*\* |
| [Dav4](dav4-dasv4-series.md) |230 - 260** | 2:1 |
| [Dasv4](dav4-dasv4-series.md) |230 - 260** | 2:1 |
| [Dv4](dv4-dsv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Dsv4](dv4-dsv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Ddv4](ddv4-ddsv4-series.md) | 195 -210* | 2:1\*\*\* |
| [Ddsv4](ddv4-ddsv4-series.md) | 195 - 210* | 2:1\*\*\* |
| [E_v3](ev3-esv3-series.md) |160 - 190* | 2:1\*\*\*|
| [Es_v3](ev3-esv3-series.md) |160 - 190* | 2:1\*\*\* |
| [Eav4](eav4-easv4-series.md) |230 - 260** | 2:1 |
| [Easv4](eav4-easv4-series.md) | 230 - 260** | 2:1 |
| [Ev4](ev4-esv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Esv4](ev4-esv4-series.md) | 195 - 210 | 2:1\*\*\* |
| [Edv4](edv4-edsv4-series.md) | 195 - 210* | 2:1\*\*\* |
| [Edsv4](edv4-edsv4-series.md) | 195 - 210* | 2:1\*\*\* |
| [F2s_v2 - F72s_v2](fsv2-series.md) |195 - 210* | 2:1\*\*\* |
| [F1 - F16](sizes-previous-gen.md) |210 - 250* | 1:1 |
| [F1s - F16s](sizes-previous-gen.md) |210 - 250* | 1:1 |
| [G1 - G5](sizes-previous-gen.md) |180 - 240* | 1:1 |
| [GS1 - GS5](sizes-previous-gen.md) |180 - 240* | 1:1 |
| [H](h-series.md) |290 - 300* | 1:1 |
| [HB](hb-series.md) |199 - 216** | 1:1 |
| [HC](hc-series.md) |297 - 315* | 1:1 |
| [L4s - L32s](sizes-previous-gen.md) |180 - 240* | 1:1 |
| [L8s_v2 - L80s_v2](lsv2-series.md) |150 - 175** | 2:1 |
| [M](m-series.md) | 160 - 180 | 2:1\*\*\* |
| [NVv4](nvv4-series.md) |230 - 260** | 2:1 |

Voici des liens vers des informations complémentaires sur les différentes tailles :

- [À usage général](sizes-general.md)
- [Mémoire optimisée](sizes-memory.md)
- [Optimisé pour le calcul](sizes-compute.md)
- [Optimisé pour le GPU](sizes-gpu.md)
- [Calcul haute performance](sizes-hpc.md)
- [Optimisé pour le stockage](sizes-storage.md)
