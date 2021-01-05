---
title: Séries Edv4 et Edsv4
description: Spécifications relatives aux machines virtuelles des séries Ev4, Edv4, Esv4 et Edsv4.
author: brbell
ms.author: brbell
ms.reviewer: jushiman
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/04/2020
ms.openlocfilehash: d21c75218fdd43ffcf977a1d70bc325ce5154919
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96606937"
---
# <a name="edv4-and-edsv4-series"></a>Séries Edv4 et Edsv4

Les séries Edv4 et Edsv4 s’exécutent sur les processeurs Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake) dans une configuration hyper-thread. Idéales pour les applications d’entreprise gourmandes en mémoire, elles proposent jusqu’à 504 Gio de RAM, la [technologie Intel&reg; Turbo Boost 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html), la [technologie Intel&reg; Hyper-Threading](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html) et la [technologie Intel&reg; Advanced Vector Extensions 512 (Intel&reg; AVX-512)](https://www.intel.com/content/www/us/en/architecture-and-technology/avx-512-overview.html). Elles prennent également en charge la technologie [Intel&reg; Deep Learning Boost](https://software.intel.com/content/www/us/en/develop/topics/ai/deep-learning-boost.html). Ces nouvelles tailles de machines virtuelles disposeront d’un stockage local 50 % plus volumineux ainsi que de meilleures IOPS de disque local en lecture et en écriture par rapport aux tailles [Ev3/Esv3](./ev3-esv3-series.md) avec des [machines virtuelles Gen2](./generation-2.md). Elles sont dotées d’une vitesse d’horloge de Turbo cœur de 3,4 GHz. 

## <a name="edv4-series"></a>Série Edv4

Les tailles de la série Edv4 s’exécutent sur les processeurs Intel &reg;​​Xeon&reg; Platinum 8272CL (Cascade Lake). Les tailles de machine virtuelle Edv4 proposent jusqu’à 504 Gio de RAM, en plus du stockage SSD local rapide et volumineux (jusqu’à 2 400 Gio). Ces machines virtuelles sont idéales pour les applications d’entreprise gourmandes en mémoire et les applications qui bénéficient d’un stockage local à faible latence et à haut débit. Vous pouvez associer les stockages de disque SSD Standard et HDD Standard aux machines virtuelles Edv4. 

[ACU](acu.md) : 195 - 210<br>
[Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Générations 1 et 2<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | <sup>**</sup> Débit de stockage temporaire et mis en cache max. : IOPS/Mbits/s | Nombre max de cartes réseau|Bande passante réseau attendue (Mbits/s) |
|---|---|---|---|---|---|---|---|
| Standard_E2d_v4  | 2 | 16 | 75 | 4 | 19000/120 | 2|1 000 |
| Standard_E4d_v4  | 4 | 32 | 150 | 8 | 38500/242 | 2|2000 |
| Standard_E8d_v4 | 8 | 64 | 300 | 16 | 77000/485 | 4|4000 |
| Standard_E16d_v4 | 16 | 128 | 600 | 32 | 154000/968 | 8|8000 |
| Standard_E20d_v4 | 20 | 160 | 750 | 32 | 193000/1211  | 8|10000 |
| Standard_E32d_v4 | 32 | 256 | 1200 | 32 | 308000/1936 | 8|16000 |
| Standard_E48d_v4 | 48 | 384 | 1800 | 32 | 462000/2904 | 8|24 000 |
| Standard_E64d_v4 | 64 | 504 | 2 400 | 32 | 615000/3872 | 8|30000 |


<sup>**</sup> Ces valeurs IOPS peuvent être garanties avec des [machines virtuelles de deuxième génération](generation-2.md)

## <a name="edsv4-series"></a>Série Edsv4

Les tailles de la série Edsv4 s’exécutent sur les processeurs Intel &reg;​​Xeon&reg; Platinum 8272CL (Cascade Lake). Les tailles de machine virtuelle Edsv4 proposent jusqu’à 504 Gio de RAM, en plus du stockage SSD local rapide et volumineux (jusqu’à 2 400 Gio). Ces machines virtuelles sont idéales pour les applications d’entreprise gourmandes en mémoire et les applications qui bénéficient d’un stockage local à faible latence et à haut débit.

[ACU](acu.md) : 195-210<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Pris(e) en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Pris(e) en charge<br>
[Prise en charge de la génération de machine virtuelle](generation-2.md) : Générations 1 et 2<br>
<br>

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | <sup>**</sup> Débit de stockage temporaire et mis en cache max. : IOPS/Mbits/s (taille du cache en Gio) | Débit du disque non mis en cache max. : IOPS/Mbits/s | Nombre max de cartes réseau|Bande passante réseau attendue (Mbits/s) |
|---|---|---|---|---|---|---|---|---|
| Standard_E2ds_v4  | 2 | 16 | 75 | 4 | 19000/120(50) | 3 200/48 | 2|1 000 |
| Standard_E4ds_v4  | 4 | 32 | 150 | 8 | 38500/242(100) | 6 400/96 | 2|2000 |
| Standard_E8ds_v4 | 8 | 64 | 300 | 16 | 77000/485(200) | 12 800/192 | 4|4000 |
| Standard_E16ds_v4 | 16 | 128 | 600 | 32 | 154000/968(400) | 25 600/384 | 8|8000 |
| Standard_E20ds_v4 | 20 | 160 | 750 | 32 | 193000/1211(500)  | 32 000/480  | 8|10000 |
| Standard_E32ds_v4 | 32 | 256 | 1200 | 32 | 308000/1936(800) | 51 200/768  | 8|16000 |
| Standard_E48ds_v4 | 48 | 384 | 1800 | 32 | 462000/2904(1200) | 76 800/1152 | 8|24 000 |
| Standard_E64ds_v4 <sup>1</sup> | 64 | 504 | 2 400 | 32 | 615000/3872(1600) | 80 000/1 200 | 8|30000 |
| Standard_E80ids_v4 <sup>2</sup> | 80 | 504 | 2 400 | 32 | 615000/3872(1600) | 80 000/1 200 | 8|30000 |

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
