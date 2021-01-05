---
title: Série NCas T4 v3
description: Spécifications pour les machines virtuelles de la série NCas T4 v3.
services: virtual-machines
ms.subservice: sizes
author: vikancha-MSFT
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 08/10/2020
ms.author: vikancha
ms.openlocfilehash: edbd8b2fefb90bb95bfcd08c126a42c810ad081b
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511596"
---
# <a name="ncast4_v3-series-in-preview"></a>Série NCasT4_v3 (en préversion) 

Les machines virtuelles de la série NCasT4_v3 sont alimentées par [processeurs graphiques NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) et des processeurs AMD EPYC 7V12(Rome). Les machines virtuelles comportent jusqu’à 4 processeurs graphiques NVIDIA T4 avec 16 Go de mémoire chacun, jusqu’à 64 cœurs AMD EPYC 7V12 (Rome) et 440 Gio de mémoire système. Ces machines virtuelles sont parfaites pour le déploiement de services d’intelligence artificielle, tels que l’inférence en temps réel des demandes générées par l’utilisateur, ou pour les charges de travail graphiques et de visualisation interactives à l’aide du pilote de grille NVIDIA et de la technologie GPU virtuelle. Les charges de travail de calcul standard GPU basées sur CUDA, TensorRT, Caffe, ONNX et d’autres infrastructures, ou sur des applications graphiques à accélération GPU basées sur OpenGL et DirectX peuvent être déployées de manière économique, avec une proximité étroite pour les utilisateurs, sur la série NCasT4_v3.

> [!NOTe]
> [Soumettez une demande](https://aka.ms/NCT4v3Preview) pour participer au programme de préversion.

<br>

[ACU](acu.md) : 230-260<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Non pris en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Non pris en charge<br>
[Génération de machine virtuelle prise en charge](generation-2.md) : Génération 1<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | GPU | Mémoire GPU : Gio | Disques de données max. | Nombre max de cartes réseau / Bande passante réseau attendue (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NC4as_T4_v3 |4 |28 |180 | 1 | 16 | 8 | 2 / 8000 |
| Standard_NC8as_T4_v3 |8 |56 |360 | 1 | 16 | 16 | 4 / 8 000  |
| Standard_NC16as_T4_v3 |16 |110 |360 | 1 | 16 | 32 | 8 / 8000  |
| Standard_NC64as_T4_v3 |64 |440 |2880 | 4 | 64 | 32 | 8 / 32000  |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Systèmes d’exploitation et pilotes pris en charge

Pour tirer parti des fonctionnalités GPU des machines virtuelles de série NCasT4_V3 Azure sous Windows ou Linus, des pilotes graphiques NVIDIA doivent être installés.

Pour installer manuellement les pilotes graphiques NVIDIA, consultez [Installer les pilotes GPU NVIDIA sur les machines virtuelles de série N exécutant Windows](./windows/n-series-driver-setup.md) pour connaître les systèmes d’exploitation pris en charge, les pilotes et les étapes d’installation et de vérification.

## <a name="other-sizes"></a>Autres tailles

- [Usage général](sizes-general.md)
- [Mémoire optimisée](sizes-memory.md)
- [Optimisé pour le stockage](sizes-storage.md)
- [Optimisé pour le GPU](sizes-gpu.md)
- [Calcul haute performance](sizes-hpc.md)
- [Générations précédentes](sizes-previous-gen.md)

## <a name="next-steps"></a>Étapes suivantes

Lisez-en davantage sur les [Unités de calcul Azure (ACU)](acu.md) pour découvrir comment comparer les performances de calcul entre les références Azure.
