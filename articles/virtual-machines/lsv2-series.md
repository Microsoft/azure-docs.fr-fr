---
title: Série Lsv2 – Machines virtuelles Microsoft Azure
description: Spécifications pour les machines virtuelles de la série Lsv2.
author: sasha-melamed
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: ba971919037230cb6c09e627fde448647063b5b9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498419"
---
# <a name="lsv2-series"></a>Série Lsv2

La série Lsv2 offre un stockage NVMe local directement mappé, à haut débit et faible latence. Elle est dotée du [processeur AMD EPYC<sup>TM</sup> 7551](https://www.amd.com/en/products/epyc-7000-series) disposant d’une augmentation de la fréquence de tous les cœurs de 2,55 GHz et d’une augmentation maximale de 3 GHz. Les machines virtuelles de la série Lsv2 comprennent entre 8 et 80 processeurs virtuels dans une configuration de multithreading simultané.  Il existe 8 Gio de mémoire par processeur virtuel et un périphérique NVMe SSD M.2 de 1,92 To pour 8 processeurs virtuels, avec une limite maximale de 19,2 To (10 x 1,92 To) sur le L80s v2.

> [!NOTE]
> Les machines virtuelles de la série Lsv2 sont optimisées pour utiliser le disque local du nœud attaché directement à la machine virtuelle au lieu d’utiliser des disques de données durables. Cela permet d’augmenter les IOPS/le débit de vos charges de travail. Les séries Lsv2 et Ls ne prennent pas en charge la création d’un cache local pour augmenter le taux d’IOPS pouvant être atteint par les disques de données durables.
>
> Le débit élevé et les IOPS du disque local rendent les machines virtuelles des séries Lsv2 idéales pour les magasins NoSQL tels qu’Apache Cassandra et MongoDB, qui répliquent les données sur plusieurs machines virtuelles pour assurer une persistance en cas de défaillance d’une seule machine virtuelle.
>
> Pour en savoir plus, consultez Optimiser les performances sur les machines virtuelles de la série Lsv2 pour [Windows](../virtual-machines/windows/storage-performance.md) ou [Linux](../virtual-machines/linux/storage-performance.md).  

[ACU](acu.md) : 150-175<br>
[Stockage Premium](premium-storage-performance.md) : Pris(e) en charge<br>
[Mise en cache du Stockage Premium](premium-storage-performance.md) : Non pris en charge<br>
[Migration dynamique](maintenance-and-updates.md) : Non pris en charge<br>
[Mises à jour avec préservation de la mémoire](maintenance-and-updates.md) : Non pris en charge<br>
[Génération de machine virtuelle prise en charge](generation-2.md) : Génération 1 et 2<br>
Mode rafale : Prise en charge<br>
<br>

| Taille | Processeurs virtuels | Mémoire (Gio) | Disque temporaire<sup>1</sup> (Gio) | Disques NVMe<sup>2</sup> | Débit de disque NVMe <sup>3</sup> (IOPS de lecture/Mbits/s) | Débit de disque de données sans mise en cache (IOPS/Mbits/s)<sup>4</sup> | Débit maximal de disque de données sans mise en cache avec mode rafale (IOPS/Mbits/s)<sup>5</sup>| Nombre max de disques de données | Nombre max de cartes réseau | Bande passante réseau attendue (Mbit/s) |
|---|---|---|---|---|---|---|---|---|---|---|
| Standard_L8s_v2   |  8 |  64 |  80 |  1 x 1,92 To  | 400 000/2 000  | 8 000/160   | 8000/1280 | 16 | 2 | 3200   |
| Standard_L16s_v2  | 16 | 128 | 160 |  2 x 1,92 To  | 800 000/4 000  | 16 000/320  | 16000/1280 | 32 | 4 | 6 400   |
| Standard_L32s_v2  | 32 | 256 | 320 |  4 x 1,92 To  | 1,5 M/8 000    | 32 000/640  | 32000/1280 | 32 | 8 | 12800  |
| Standard_L48s_v2  | 48 | 384 | 480 |  6 x 1,92 To  | 2,2 M/14 000   | 48000/960  | 48000/2000 | 32 | 8 | 16000+ |
| Standard_L64s_v2  | 64 | 512 | 640 |  8 x 1,92 To  | 2,9 M/16 000   | 64 000/1 280 | 64000/2000 | 32 | 8 | 16000+ |
| Standard_L80s_v2<sup>6</sup> | 80 | 640 | 800 | 10 x 1,92 To | 3,8 M / 20 000 | 80000/1400 | 80 000/2 000 | 32 | 8 | 16000+ |

<sup>1</sup> Les machines virtuelles de la série Lsv2 disposent d’un disque de ressources temporaire SCSI standard pour l’utilisation du fichier d’échange du système d’exploitation (D: sur Windows, /dev/sdb sur Linux). Ce disque offre un stockage de 80 Gio, 4 000 IOPS et un taux de transfert de 80 Mbits/s pour 8 processeurs virtuels (par exemple, Standard_L80s_v2 fournit 800 Gio à 40 000 IOPS et 800 Mbits/s). Ainsi, les lecteurs NVMe peuvent être entièrement dédiés à l’utilisation d’applications. Ce disque est éphémère, et toutes les données seront perdues lors de l'arrêt/la libération.

<sup>2</sup> Les disques NVMe locaux sont éphémères ; les données présentes sur ces disques seront perdues si vous arrêtez/libérez votre machine virtuelle.

<sup>3</sup> La technologie Hyper-V NVMe Direct fournit un accès illimité aux disques NVMe locaux mappés de manière sécurisée à l'espace de la machine virtuelle invitée.  Pour atteindre des performances maximales, vous devez utiliser la dernière build WS2019, ou Ubuntu 18.04 ou 16.04, de la Place de marché Azure.  Les performances en écriture varient en fonction de la taille des E/S, de la charge du lecteur et de l’utilisation des fonctionnalités.

<sup>4</sup> Les machines virtuelles de la série Lsv2 ne fournissent pas de cache hôte pour le disque de données, car cela ne profite pas aux charges de travail Lsv2.

<sup>5</sup> Les machines virtuelles de la série Lsv2 peuvent [augmenter via le mode rafale](./disk-bursting.md) leurs performances de disque pendant 30 minutes à la fois. 

<sup>6</sup> Les machines virtuelles avec plus de 64 processeurs virtuels nécessitent l’un de ces systèmes d’exploitation invités pris en charge :

- Windows Server 2016 ou version ultérieure
- Ubuntu 16.04 LTS or version ultérieure, avec noyau Azure adapté (noyau 4.15 ou version ultérieure)
- SLES 12 SP2 ou version ultérieure
- RHEL ou CentOS version 6.7 à 6.10, avec le package LIS fourni par Microsoft 4.3.1 (ou version ultérieure) installé
- RHEL ou CentOS version 7.3, avec le package LIS fourni par Microsoft 4.2.1 (ou version ultérieure) installé
- RHEL ou CentOS 7.6 ou version ultérieure
- Oracle Linux avec UEK4 ou version ultérieure
- Debian 9 avec le noyau backports, Debian 10 ou version ultérieure
- CoreOS avec un noyau 4.14 ou version ultérieure

## <a name="size-table-definitions"></a>Définitions des tailles de tables

- La capacité de stockage est indiquée en unités de Gio ou 1 024^3 octets. Lors de la comparaison de disques mesurés en Go (1 000^3 octets) à des disques mesurés en Gio (1 024^3) n’oubliez pas que les indications de capacité en Gio peuvent sembler plus petites. Par exemple, 1 023 Gio = 1 098,4 Go
- Le débit de disque est mesuré en opérations d’entrée/sortie par seconde (IOPS) et Mbit/s  où Mbit/s  = 10^6 octets par seconde.
- Si vous souhaitez obtenir des performances optimales pour vos machines virtuelles, vous devez limiter le nombre de disques de données à 2 disques par processeur virtuel.
- La **bande passante réseau attendue** est la [bande passante agrégée maximale qui est allouée par type de machine virtuelle](../virtual-network/virtual-machine-network-throughput.md) entre toutes les cartes réseau, pour toutes les destinations. Les limites supérieures ne sont pas garanties, mais servent de points de repère pour sélectionner le type de machine virtuelle adapté à l’application prévue. Les performances réseau réelles dépendent de nombreux facteurs, notamment la congestion du réseau, les charges de l’application, ainsi que les paramètres réseau. Pour plus d’informations sur l’optimisation du débit du réseau, consultez [Optimisation du débit du réseau pour Windows et Linux](../virtual-network/virtual-network-optimize-network-bandwidth.md). Pour atteindre la performance réseau attendue sous Linux ou Windows, il peut être nécessaire de sélectionner une version spécifique ou d’optimiser votre machine virtuelle. Pour plus d’informations, consultez [Tester de manière fiable le débit d’une machine virtuelle](../virtual-network/virtual-network-bandwidth-testing.md).


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