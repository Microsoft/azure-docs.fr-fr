---
title: Fichier include
description: Fichier include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 01/22/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: c0627dd0833e3b20468eb5f50fbeb9fd9d9ae2b3
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2020
ms.locfileid: "88864948"
---
**Transferts de données sortants** : [transferts de données sortants](https://azure.microsoft.com/pricing/details/bandwidth/) (données sortant des centres de données Azure) sont facturés en fonction de la bande passante utilisée.

**Transactions** : vous êtes facturé pour le nombre de transactions effectuées sur un disque managé standard. Pour les disques SSD Standard, chaque opération d’E/S inférieure ou égale à 256 Kio de débit est considérée comme une seule opération d’E/S. Les opérations d’E/S dont le débit est supérieur à 256 Kio sont considérées comme des opérations d’E/S distinctes, de 256 Kio chacune. Pour les disques SSD Standard, chaque opération d’E/S est considérée comme une seule transaction, indépendamment de la taille des E/S.

Pour plus d’informations sur la tarification d’Azure Disques managés (dont les coûts de transaction), consultez la page [Tarification de la fonctionnalité Disques managés](https://azure.microsoft.com/pricing/details/managed-disks).

### <a name="ultra-disk-vm-reservation-fee"></a>Frais de réservation de machine virtuelle avec disque Ultra

Les machines virtuelles Azure peuvent indiquer si elles sont compatibles avec les disques Ultra. Une machine virtuelle compatible avec les disques Ultra alloue une capacité de bande passante dédiée entre l'instance de machine virtuelle de calcul et l'unité d'échelle de stockage de blocs pour optimiser les performances et réduire la latence. L'ajout de cette fonctionnalité sur la machine virtuelle entraîne des frais de réservation qui ne sont imposés que si vous avez activé la fonctionnalité de disque Ultra sur la machine virtuelle sans y joindre de disque Ultra. Lorsqu'un disque Ultra est joint à la machine virtuelle compatible avec les disques Ultra, ces frais ne s'appliquent pas. Les frais sont calculés par processeur virtuel provisionné sur la machine virtuelle. 

> [!Note]
> Pour les [tailles de machine virtuelle avec des cœurs restreints](../articles/virtual-machines/constrained-vcpu.md), les frais de réservation sont basés sur le nombre réel de processeurs virtuels et non sur les cœurs restreints. Pour Standard_E32-8s_v3, les frais de réservation sont basés sur 32 cœurs. 

Pour plus d’informations sur la tarification des disques Ultra, consultez la [page relative à la tarification des disques Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

### <a name="azure-disk-reservation"></a>Réservation de disque Azure

La réservation de disque est la possibilité d’acheter à l’avance un an de stockage sur disque avec une remise, ce qui réduit le coût total pour vous. Lors de l’achat d’une réservation de disque, vous sélectionnez une référence SKU de disque spécifique dans une région cible, par exemple 10 disques SSD P30 (1 Tio) Premium dans la région USA Est 2 pour une période d’un an. L’expérience de réservation est similaire aux instances de machines virtuelles réservées. Vous pouvez regrouper les réservations de machines virtuelles et de disques pour optimiser vos économies. Pour le moment, la réservation de disques Azure offre un plan d’engagement d’un an pour les références (SKU) de disques SSD Premium de P30 (1 Tio) à P80 (32 Tio) dans toutes les régions de production. Pour plus d’informations sur la tarification des disques réservés, consultez [Page de tarification des disques Azure](https://azure.microsoft.com/pricing/details/managed-disks/).