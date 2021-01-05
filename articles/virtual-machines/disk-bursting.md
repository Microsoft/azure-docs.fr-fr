---
title: Mode rafale des disques managés
description: Découvrez le bursting de disque pour les disques Azure et les machines virtuelles Azure.
author: albecker1
ms.author: albecker
ms.date: 09/22/2020
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: a8d1a69070628248fb710a614369ff9311252c05
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93102645"
---
# <a name="managed-disk-bursting"></a>Mode rafale des disques managés
[!INCLUDE [managed-disks-bursting](../../includes/managed-disks-bursting.md)]

## <a name="virtual-machine-level-bursting"></a>Mode rafale au niveau des machines virtuelles
La prise en charge du mode rafale au niveau des machines virtuelles est activée dans toutes les régions du cloud public sur les tailles prises en charge suivantes : 
- [Série Lsv2](lsv2-series.md)

Le mode rafale au niveau des machines virtuelles est également disponible dans USA Centre-Ouest pour les tailles prises en charge suivantes :
- [Dsv3-series](dv3-dsv3-series.md)
- [Série Esv3](ev3-esv3-series.md)

Le mode rafale est activé par défaut pour les machines virtuelles qui le prennent en charge.

## <a name="disk-level-bursting"></a>Mode rafale au niveau des disques
Le mode rafale est également disponible sur nos [SSD Premium](disks-types.md#premium-ssd) pour les tailles de disque P20 et inférieures dans toutes les régions des clouds Azure publics, gouvernementaux et en Chine. Le mode rafale du disque est activé par défaut sur tous les déploiements nouveaux et existants des tailles de disque qui le prennent en charge. 

[!INCLUDE [managed-disks-bursting](../../includes/managed-disks-bursting-2.md)]