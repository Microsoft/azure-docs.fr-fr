---
title: Séries Ev4 et Esv4 – Azure Virtual Machines
description: Spécifications pour les machines virtuelles des séries Ev4 et Esv4.
author: brbell
ms.author: brbell
ms.reviewer: cynthn
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 6/8/2020
ms.openlocfilehash: 38020b38ebd879bf19c243c2055e73198e9fcc7c
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96606920"
---
# <a name="ev4-and-esv4-series"></a>Séries Ev4 et Esv4

Les séries Ev4 et Esv4 s’exécutent sur les processeurs Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake) dans une configuration hyper-thread. Elles sont idéales pour diverses applications d’entreprise utilisant beaucoup de mémoire et offrent jusqu’à 504 Gio de RAM. Elles sont dotées d’une vitesse d’horloge de Turbo cœur de 3,4 GHz.

> [!NOTE]
> Pour accéder à la FAQ, consultez [Tailles de machines virtuelles Azure sans disque temporaire local](azure-vms-no-temp-disk.md).

## <a name="ev4-series"></a>Série Ev4

Les tailles de la série Ev4 s’exécutent sur Intel Xeon&reg; Platinum 8272CL (Cascade Lake). Ces instances sont idéales pour les applications d’entreprise utilisant beaucoup de mémoire. Les machines virtuelles de cette série sont dotées de la technologie Intel&reg; Hyper-Threading.

Le stockage sur disque de données à distance est facturé séparément des machines virtuelles. Pour utiliser des disques Stockage Premium, choisissez les tailles Esv4. Les tarifs et compteurs de facturation de ces tailles sont identiques à ceux de la série Ev4.

[ACU](acu.md) : 195 - 210<br>
[Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Génération 1<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Nombre max de cartes réseau|Bande passante réseau attendue (Mbits/s) |
|---|---|---|---|---|---|---|
| Standard_E2_v4  | 2 | 16   | Stockage distant uniquement | 4 | 2|1 000  |
| Standard_E4_v4  | 4 | 32  | Stockage distant uniquement | 8 | 2|2000  |
| Standard_E8_v4  | 8 | 64 | Stockage distant uniquement | 16 | 4|4000 |
| Standard_E16_v4 | 16 | 128 | Stockage distant uniquement | 32 | 8|8000 |
| Standard_E20_v4 | 20 | 160 | Stockage distant uniquement | 32 | 8|10000 |
| Standard_E32_v4 | 32 | 256 | Stockage distant uniquement | 32 | 8|16000 |
| Standard_E48_v4 | 48 | 384 | Stockage distant uniquement | 32 | 8|24 000 |
| Standard_E64_v4 | 64 | 504 | Stockage distant uniquement | 32| 8|30000 |


## <a name="esv4-series"></a>Série Esv4

Les tailles de la série Esv4 s’exécutent sur Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake). Ces instances sont idéales pour les applications d’entreprise utilisant beaucoup de mémoire. Les machines virtuelles de cette série sont dotées de la technologie Intel&reg; Hyper-Threading. Le stockage sur disque de données à distance est facturé séparément des machines virtuelles.

[ACU](acu.md) : 195-210<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Générations 1 et 2<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit du disque non mis en cache max. : IOPS/Mbits/s | Nombre max de cartes réseau|Bande passante réseau attendue (Mbits/s) |
|---|---|---|---|---|---|---|---|
| Standard_E2s_v4  | 2 | 16  | Stockage distant uniquement | 4 | 3 200/48 | 2|1 000  |
| Standard_E4s_v4  | 4 | 32  | Stockage distant uniquement | 8 | 6 400/96 | 2|2000  |
| Standard_E8s_v4  | 8 | 64  | Stockage distant uniquement | 16 | 12 800/192 | 4|4000 |
| Standard_E16s_v4 | 16 | 128 | Stockage distant uniquement | 32 | 25 600/384 | 8|8000 |
| Standard_E20s_v4 | 20 | 160 | Stockage distant uniquement | 32 | 32 000/480  | 8|10000 |
| Standard_E32s_v4 | 32 | 256 | Stockage distant uniquement | 32 | 51 200/768  | 8|16000 |
| Standard_E48s_v4 | 48 | 384 | Stockage distant uniquement | 32 | 76 800/1152 | 8|24 000 |
| Standard_E64s_v4<sup>1</sup> | 64 | 504| Stockage distant uniquement | 32 | 80 000/1 200 | 8|30000 |
| Standard_E80is_v4 <sup>2</sup> | 80 | 504 | Stockage distant uniquement | 32 | 80 000/1 200 | 8|30000 |

<sup>1</sup> [Tailles avec nombre de cœurs limité disponibles](./constrained-vcpu.md).

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
