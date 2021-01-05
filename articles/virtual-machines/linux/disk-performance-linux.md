---
title: Performances des disques et des machines virtuelles
description: Découvrir plus d’informations sur la façon dont les machines virtuelles et leurs disques attachés fonctionnent en combinaison pour les meilleures performances
author: albecker1
ms.author: albecker
ms.date: 10/12/2020
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 092368bb66784a00d5116da0b6be6513f8ebb261
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518057"
---
# <a name="virtual-machine-and-disk-performance"></a>Performances des disques et des machines virtuelles
[!INCLUDE [VM and Disk Performance](../../../includes/virtual-machine-disk-performance.md)]

## <a name="virtual-machine-uncached-vs-cached-limits"></a>Limites des machines virtuelles sans mise en cache et avec mise en cache
Les machines virtuelles pour lesquelles le stockage Premium et la mise en cache du stockage Premium sont activés présentent deux limites différentes en matière de bande passante de stockage. Examinons la machine virtuelle Standard_D8s_v3 à titre d’exemple. Voici la documentation relative à la [série Dsv3](../dv3-dsv3-series.md) et à la machine virtuelle Standard_D8s_v3 :

[!INCLUDE [VM and Disk Performance](../../../includes/virtual-machine-disk-performance-2.md)]

Exécutons un test d’évaluation sur cette combinaison de machine virtuelle et de disque qui crée une activité d’E/S. Pour savoir comment créer un benchmark des E/S de stockage sur Azure, consultez [Créer un benchmark de votre application sur Stockage sur disque Azure](disks-benchmarks.md). À partir de l’outil d’évaluation, vous pouvez voir que la combinaison de machines virtuelles et de disques peut atteindre 22 800 IOPS :

[!INCLUDE [VM and Disk Performance](../../../includes/virtual-machine-disk-performance-3.md)]
