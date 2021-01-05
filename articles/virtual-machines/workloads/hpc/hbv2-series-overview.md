---
title: Vue d’ensemble des machines virtuelles de série HBv2 – Machines virtuelles Azure | Microsoft Docs
description: En savoir plus sur la taille des machines virtuelles de la série HBv2 dans Azure.
services: virtual-machines
author: vermagit
tags: azure-resource-manager
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure-services
ms.topic: article
ms.date: 09/28/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: be66f29881250843f70ba85b8ef7c80ae8b31aa6
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966936"
---
# <a name="hbv2-series-virtual-machine-overview"></a>Vue d’ensemble des machines virtuelles de la série HBv2 

 
L’optimisation des performances des applications de calcul haute performance (HPC)sur AMD EPYC nécessite une approche bien pensée de l’emplacement de la mémoire et du placement du processus. Nous décrivons ci-dessous l’architecture AMD EPYC et son implémentation sur Azure pour les applications HPC. Nous utiliserons le terme **pNUMA** pour faire référence à un domaine NUMA physique et **vNUMA** pour faire référence à un domaine NUMA virtualisé. 

Physiquement, un serveur de la [série HBv2](../../hbv2-series.md) a 2 processeurs EPYC 7742 de 64 cœurs, soit un total de 128 cœurs physiques. Ces 128 cœurs sont divisées en 32 domaines pNUMA (16 par socket), chacun d’eux comprenant 4 cœurs et étant appelé **complexe de cœurs** (ou **CCX**) par AMD. Chaque CCX a son propre cache L3, qui représente la limite pNUMA/vNUMA du système d’exploitation. Quatre CCX adjacents partagent l’accès à deux canaux de DRAM physique. 

Pour permettre à l’hyperviseur Azure de fonctionner sans interférer avec la machine virtuelle, nous réservons les domaines pNUMA physiques 0 et 16 (c’est-à-dire le premier CCX de chaque socket de processeur). Les 30 domaines pNUMA restants sont tous attribués à la machine virtuelle et deviennent alors des vNUMA. Par conséquent, la machine virtuelle verra :

`(30 vNUMA domains) * (4 cores/vNUMA) = 120` cœurs par machine virtuelle 

La machine virtuelle elle-même ne sait pas que les pNUMA 0 et 16 sont réservés. Elle énumère les vNUMA qu’elle considère aller de 0 à 29, avec 15 vNUMA par socket distribués de manière symétrique : les vNUMA 0 à 14 sur vSocket 0 et les vNUMA 15 à 29 sur vSocket 1. Dans la section suivante, vous trouverez des instructions sur la meilleure façon d’exécuter des applications MPI sur cette disposition NUMA asymétrique. 

L’épinglage de processus fonctionne sur les machines virtuelles de série HBv2, car nous exposons le silicium sous-jacent tel quel à la machine virtuelle invitée. Nous recommandons vivement l’épinglage de processus (process pinning) à des fins de performances optimales. 


## <a name="hardware-specifications"></a>Spécifications matérielles 

| Spécifications matérielles          | Machine virtuelle de série HBv2                   | 
|----------------------------------|----------------------------------|
| Cœurs                            | 120 (SMT désactivé)               | 
| UC                              | AMD EPYC 7742                    | 
| Fréquence de l’UC (non AVX)          | Environ 3,1 GHz (simple + tous les cœurs)    | 
| Mémoire                           | 4 Go/cœur (480 Go au total)         | 
| Disque local                       | Disque NVMe de 960 Go (bloc), SSD de 480 Go (fichier d’échange) | 
| Infiniband                       | EDR Mellanox ConnectX-6 de 200 Gbits | 
| Réseau                          | Ethernet 50 Gbits (40 Gbits utilisables) SmartNIC Azure 2ème gén. | 


## <a name="software-specifications"></a>Spécifications logicielles 

| Spécifications logicielles     | Machine virtuelle de série HBv2                                            | 
|-----------------------------|-----------------------------------------------------------|
| Taille de travail MPI max            | 36 000 cœurs (300 machines virtuelles dans un seul groupe de machines virtuelles identiques avec singlePlacementGroup=true) |
| Prise en charge MPI                 | HPC-X, Intel MPI, OpenMPI, MVAPICH2, MPICH, Platform MPI  |
| Frameworks supplémentaires       | Unified Communication X, libfabric, PGAS                  |
| Prise en charge de Stockage Azure       | Disques Standard et Premium (maximum 8 disques)              |
| Prise en charge du système d’exploitation pour SRIOV RDMA   | CentOS/RHEL 7.6+, SLES 12 SP4+, WinServer 2016+           |
| Prise en charge d’Orchestrator        | CycleCloud, Batch                                         | 


## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus, consultez [Architecture AMD EPYC](https://bit.ly/2Epv3kC) et [Architectures multiprocesseur](https://bit.ly/2GpQIMb). Pour plus d’informations, consultez le [Guide d’optimisation HPC pour les processeurs AMD EPYC](https://bit.ly/2T3AWZ9).
- Découvrez des informations sur les dernières annonces et des exemples HPC dans les [blogs de la communauté Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Pour une vision plus globale de l’architecture d’exécution des charges de travail HPC, consultez [Calcul haute performance (HPC) sur Azure](/azure/architecture/topics/high-performance-computing/).