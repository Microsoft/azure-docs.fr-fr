---
title: Vue d’ensemble des machines virtuelles série HB - Machines virtuelles Azure | Microsoft Docs
description: En savoir plus sur la prise en charge de la préversion de la taille de machine virtuelle série HB dans Azure.
author: vermagit
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 08/19/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 1e37f13bc6e819079c63c583a1eeda12af5e213c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965457"
---
# <a name="hb-series-virtual-machines-overview"></a>Vue d’ensemble des machines virtuelles série HB

L’optimisation des performances des applications de calcul haute performance (HPC)sur AMD EPYC nécessite une approche bien pensée de l’emplacement de la mémoire et du placement du processus. Nous décrivons ci-dessous l’architecture AMD EPYC et son implémentation sur Azure pour les applications HPC. Nous utiliserons le terme « pNUMA » pour faire référence à un domaine NUMA physique et « vNUMA » pour faire référence à un domaine NUMA virtualisé.

Physiquement, un serveur de la [série HB](../../hb-series.md) a 2 UC EPYC 7551 de 32 cœurs, soit un total de 64 cœurs physiques. Ces 64 cœurs sont divisées en 16 domaines pNUMA (8 par socket), chacun d’eux comprenant quatre cœurs et étant appelé « Complexe UC » (ou « CCX »). Chaque CCX a son propre cache L3, qui représente la limite pNUMA/vNUMA du système d’exploitation. Une paire d’accès de CCXs adjacents se partage l’accès à deux canaux de DRAM physique (32 Go de DRAM dans les serveurs de la série HB).

Pour permettre à l’hyperviseur Azure de fonctionner sans interférer avec la machine virtuelle, nous réservons le domaine pNUMA physique 0 (le premier CCX). Ensuite, nous attribuons les domaines pNUMA 1 à 15 (les unités CCX restantes) à la machine virtuelle. La machine virtuelle verra :

`(15 vNUMA domains) * (4 cores/vNUMA) = 60` cœurs par machine virtuelle

La machine virtuelle proprement dite ne sait pas que pNUMA 0 ne le lui a pas été donné. La machine virtuelle comprend pNUMA 1 à 15 comme vNUMA 0 à 14, avec 7 vNUMA sur vSocket 0 et 8 vNUMA sur vSocket 1. Bien que cela soit asymétrique, votre système d’exploitation doit démarrer et fonctionner normalement. Plus loin dans ce guide, nous indiquons comment exécuter au mieux des applications MPI sur cette disposition NUMA asymétrique.

L’épinglage de processus fonctionne sur les machines virtuelles série HB, car nous exposons le silicium sous-jacent tel quel à la machine virtuelle invitée. Nous recommandons vivement l’épinglage de processus (process pinning) à des fins de performances optimales.

Le diagramme suivant illustre la répartition des cœurs réservés pour l’hyperviseur Azure et la machine virtuelle série HB.

![Répartition des cœurs réservés pour l’hyperviseur Azure et la machine virtuelle série HB](./media/hb-series-overview/segregation-cores.png)

## <a name="hardware-specifications"></a>Spécifications matérielles

| Spécifications matérielles                | Machine virtuelle série HB                     |
|----------------------------------|----------------------------------|
| Cœurs                            | 60 (SMT désactivé)                |
| UC                              | AMD EPYC 7551                    |
| Fréquence de l’UC (non AVX)          | ~2,55 GHz (simple + tous les cœurs)   |
| Mémoire                           | 4 GB/cœur (240 Go au total)         |
| Disque local                       | -SSD de 700 Go                       |
| Infiniband                       | 100 Gbits EDR Mellanox ConnectX-5 |
| Réseau                          | 50 Gbits Ethernet (40 Gbits utilisables) SmartNIC Azure 2ème gén. |

## <a name="software-specifications"></a>Spécifications logicielles

| Spécifications logicielles           |Machine virtuelle série HB           |
|-----------------------------|-----------------------|
| Taille de travail MPI max            | 18 000 cœurs (300 machines virtuelles dans un seul groupe de machines virtuelles identiques avec singlePlacementGroup=true)  |
| Prise en charge MPI                 | HPC-X, Intel MPI, OpenMPI, MVAPICH2, MPICH, Platform MPI  |
| Frameworks supplémentaires       | Unified Communication X, libfabric, PGAS |
| Prise en charge de Stockage Azure       | Disques Standard et Premium (maximum 4 disques) |
| Prise en charge du système d’exploitation pour SRIOV RDMA   | CentOS/RHEL 7.6+, SLES 12 SP4+, WinServer 2016+  |
| Prise en charge d’Orchestrator        | CycleCloud, Batch  |

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus, consultez [Architecture AMD EPYC](https://bit.ly/2Epv3kC) et [Architectures multiprocesseur](https://bit.ly/2GpQIMb). Pour plus d’informations, consultez le [Guide d’optimisation HPC pour les processeurs AMD EPYC](https://bit.ly/2T3AWZ9).
- Découvrez des informations sur les dernières annonces et des exemples et des résultats HPC sur les [Blogs de la communauté Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Pour une vision plus globale de l’architecture d’exécution des charges de travail HPC, consultez [Calcul haute performance (HPC) sur Azure](/azure/architecture/topics/high-performance-computing/).
