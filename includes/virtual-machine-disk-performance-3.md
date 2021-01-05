---
title: Fichier include
description: Fichier include
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 10/12/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 8eff9da82fdfa5749fd1c2bc04652d5c8ce8dfd2
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518055"
---
![Capture d’écran de la sortie f i o montrant r=22.8k mis en surbrillance.](media/vm-disk-performance/utilization-metrics-example/fio-output.jpg)

La Standard_D8s_v3 peut atteindre un total de 28 600 d’IOPS. À l’aide des métriques, nous allons examiner ce qui se passe et identifier notre goulot d’étranglement d’e/s du stockage. Dans le volet de gauche, sélectionnez **Métriques**  :

![Capture d’écran montrant les métriques mises en surbrillance dans le volet de gauche.](media/vm-disk-performance/utilization-metrics-example/metrics-menu.jpg)

Examinons tout d’abord la métrique **Pourcentage d’IOPS en cache de machine virtuelle consommées**  :

![Capture d’écran montrant le pourcentage d’IOPS consommées mises en cache sur une machine virtuelle.](media/vm-disk-performance/utilization-metrics-example/vm-cached.jpg)

Cette métrique nous informe que 61 % des 16 000 IOPS allouées aux IOPS en cache sur la machine virtuelle sont utilisées. Ce pourcentage signifie que le goulot d’étranglement d’E/S de stockage n’est pas dans les disques mis en cache, car la valeur n’est pas 100 %. Examinons à présent la métrique **Pourcentage d’IOPS non mises en cache de machine virtuelle consommées**  :

![Capture d’écran montrant le pourcentage d’IOPS consommées non mises en cache sur une machine virtuelle.](media/vm-disk-performance/utilization-metrics-example/vm-uncached.jpg)

Cette métrique est à 100 %. Elle nous indique que l’ensemble des 12 800 IOPS allouées aux IOPS non mises en cache sur la machine virtuelle sont utilisées. Pour y remédier, il est possible de définir une plus grande taille de machine virtuelle, qui permettrait de traiter les E/S supplémentaires. Avant cela, examinons le disque attaché pour voir combien d’IOPS il détecte. Examinez le disque de système d’exploitation en observant le **pourcentage d’IOPS du disque de système d’exploitation consommées**  :

![Capture d’écran montrant le pourcentage d’IOPS consommées sur le disque du système d’exploitation.](media/vm-disk-performance/utilization-metrics-example/os-disk.jpg)

Cette métrique nous indique qu’environ 90 % des 5 000 IOPS approvisionnées pour ce disque de système d’exploitation P30 sont utilisées. Ce pourcentage signifie qu’il n’y a aucun goulot d’étranglement au niveau du disque du système d’exploitation. Nous allons maintenant examiner les disques de données attachés à la machine virtuelle en observant le **pourcentage d’IOPS du disque de données consommées**  :

![Capture d’écran montrant le pourcentage d’IOPS consommées sur le disque du système d’exploitation.](media/vm-disk-performance/utilization-metrics-example/data-disks-no-splitting.jpg)

Cette métrique nous indique que le pourcentage moyen d’IOPS consommées dans l’ensemble des disques attachés est d’environ 42 %. Ce pourcentage est calculé sur la base des IOPS utilisées par les disques et qui ne sont pas délivrées à partir du cache de l’hôte. Examinons de plus près cette métrique en appliquant le *fractionnement* à ces métriques et en fractionnant par la valeur LUN :

![Capture d’écran montrant le pourcentage d’IOPS consommées sur le disque du système d’exploitation avec fractionnement.](media/vm-disk-performance/utilization-metrics-example/data-disks-splitting.jpg)

Cette métrique nous indique que les disques de données attachés à LUN 3 et 2 sont utilisés à hauteur d’environ 85 % de leurs IOPS approvisionnées. Voici un diagramme des E/S dans l’architecture de la machine virtuelle et des disques :

![Diagramme de l’exemple de métriques d’E/S de stockage.](media/vm-disk-performance/utilization-metrics-example/metrics-diagram.jpg)
