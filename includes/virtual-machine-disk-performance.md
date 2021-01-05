---
title: Fichier include
description: Fichier include
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 09/25/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 4770ac0181c64ef800aa02ba87284c8add357e36
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518054"
---
Cet article vous aide à clarifier les performances des disques et la procédure lorsque vous combinez des machines virtuelles Azure et des disques Azure. Il explique également comment vous pouvez diagnostiquer les goulots d’étranglement des E/S de disque et les modifications que vous pouvez apporter pour optimiser les performances.

## <a name="how-does-disk-performance-work"></a>Fonctionnement des performances des disques
Les machines virtuelles Azure ont des limites de performances d’IOPS et de débit qui dépendent du type et de la taille des machines virtuelles. Les disques de système d’exploitation et les disques de données peuvent être attachés à des machines virtuelles. Les disques ont leurs propres limites d’IOPS (E/S par seconde) et de débit.

Les performances de votre application sont limitées lorsqu’elle nécessite plus d’IOPS ou de débit que ce qui est alloué pour les machines virtuelles ou les disques attachés. Lorsqu’elle est limitée, l’application fonctionne avec des performances non optimales. Cela peut entraîner des conséquences négatives, comme une latence accrue. Passons en revue quelques exemples pour clarifier ce concept. Pour que ces exemples soient faciles à suivre, nous allons seulement examiner les IOPS. Mais la même logique s’applique au débit.

## <a name="disk-io-capping"></a>Limitation d’E/S disque

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde non mises en cache : 12 800
- Disque de système d’exploitation E30
  - IOPS : 500
- Deux disques de données E30 x 2
  - IOPS : 500

![Diagramme montrant l’encapsulation au niveau du disque.](media/vm-disk-performance/disk-level-throttling.jpg)

L’application en cours d’exécution sur la machine virtuelle effectue une requête qui requiert 10 000 IOPS sur la machine virtuelle. Toutes ces opérations sont autorisées par la machine virtuelle, car la machine virtuelle Standard_D8s_v3 peut exécuter jusqu’à 12 800 IOPS.

Les requêtes de 10 000 IOPS sont divisées en trois requêtes différentes sur les différents disques :

- 1 000 IOPS sur le disque du système d’exploitation.
- 4 500 IOPS sont demandées à chaque disque de données.

Tous les disques attachés sont des disques E30 et peuvent uniquement gérer des 500 IOPS. Par conséquent, ils répondent avec 500 IOPS chacun. Les performances de l’application sont limitées par les disques attachés et ne peuvent traiter que 1 500 IOPS. L’application peut fonctionner avec des performances de pointe à 10 000 IOPS si des disques plus performants sont utilisés, comme des disques SSD Premium P30.

## <a name="virtual-machine-io-capping"></a>Limitation des E/S de machine virtuelle

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde non mises en cache : 12 800
- Disque de système d’exploitation P30
  - IOPS : 5 000
- Deux disques de données P30 x 2
  - IOPS : 5 000

![Diagramme montrant l’encapsulation au niveau de la machine virtuelle.](media/vm-disk-performance/vm-level-throttling.jpg)

L’application en cours d’exécution sur la machine virtuelle effectue une requête qui requiert 15 000 IOPS. Malheureusement, la machine virtuelle Standard_D8s_v3 est configurée pour gérer au plus 12 800 IOPS. L’application est limitée par les contraintes de la machine virtuelle et doit allouer les 12 800 IOPS affectées.

Ces requêtes de 12 800 IOPS sont divisées en trois requêtes différentes sur les différents disques :

- 4 267 IOPS sur le disque du système d’exploitation.
- 4 266 IOPS sont demandées à chaque disque de données.

Tous les disques attachés sont des disques P30 qui peuvent gérer des 5 000 IOPS. Ils répondent donc aux montants demandés.
