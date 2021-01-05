---
title: Débit réseau des machines virtuelles Azure | Microsoft Docs
description: Apprenez-en davantage sur le débit réseau des machines virtuelles Azure, notamment la façon dont la bande passante est allouée à un ordinateur virtuel.
services: virtual-network
documentationcenter: na
author: steveesp
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 4/26/2019
ms.author: steveesp
ms.reviewer: kumud, mareat
ms.openlocfilehash: f0bad935c7c3d44f57dd171f714f31856bc2089c
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91361311"
---
# <a name="virtual-machine-network-bandwidth"></a>Bande passante réseau des machines virtuelles

Azure offre une variété de tailles et de types de machines virtuelles, qui présentent une combinaison différente de capacités de performances. L’une de ces capacités est la bande passante réseau (ou débit réseau), mesurée en mégabits par seconde (Mbits/s). Étant donné que les machines virtuelles sont hébergées sur du matériel partagé, la capacité du réseau doit être répartie équitablement entre les machines virtuelles partageant le même matériel. Les machines virtuelles plus volumineuses bénéficient d’une plus grande bande passante que les machines plus petites.
 
La bande passante réseau allouée à chaque machine virtuelle est mesurée en fonction du trafic sortant de la machine virtuelle. L’ensemble du trafic réseau quittant la machine virtuelle est mesuré en fonction de la limite d’allocation, quelle que soit la destination. Par exemple, si une machine virtuelle se voit allouer une limite de 1 000 Mbits/s, cette limite s’applique de toute façon, que le trafic sortant soit destiné à une machine virtuelle du réseau virtuel ou à une machine en dehors d’Azure.
 
Le trafic en entrée n’est pas compté ou limité directement. Toutefois, il existe d’autres facteurs, tels que les limites en termes de processeur et de stockage, susceptibles d’affecter la capacité d’une machine virtuelle à traiter les données entrantes.

La mise en réseau accélérée est une fonctionnalité conçue pour améliorer les performances du réseau, notamment la latence, le débit et le taux d’utilisation des processeurs. La mise en réseau accélérée peut améliorer le débit de la machine virtuelle, mais en deçà de la limite de bande passante allouée à la machine virtuelle. Pour en savoir plus sur la mise en réseau accélérée, consultez « Mise en réseau accélérée pour les machines virtuelles [Windows](create-vm-accelerated-networking-powershell.md) ou [Linux](create-vm-accelerated-networking-cli.md) ».
 
Les machines virtuelles Azure doivent être associées à une interface réseau minimum. La bande passante allouée à une machine virtuelle représente l’intégralité du trafic sortant sur l’ensemble des interfaces réseau associées à une machine virtuelle. En d’autres termes, la bande passante est allouée machine virtuelle par machine virtuelle, quel que soit le nombre d’interfaces réseau associées à cette machine. Pour savoir combien d’interfaces réseau les machines virtuelles Azure de différentes tailles peuvent prendre en charge, consultez les informations sur les tailles de machines virtuelles [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). 

## <a name="expected-network-throughput"></a>Débit réseau attendu

Le débit sortant attendu et le nombre d’interfaces réseau prises en charge par chaque taille de machine virtuelle sont détaillés dans les spécifications des tailles de machine virtuelle Azure [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Sélectionnez un type, comme le type d’usage général, puis choisissez une série de tailles dans la page qui en résulte, par exemple « Dv2-series ». Chaque série comporte une table, dont la dernière colonne, intitulée **Nombre max de cartes réseau/Performance réseau attendue (Mbits/s)** , détaille les spécifications de mise en réseau. 

Cette limite de débit s’applique à la machine virtuelle. Le débit n’est pas affecté par les facteurs suivants :
- **Nombre d’interfaces réseau** : la limite de bande passante inclut l’ensemble du trafic sortant en provenance de la machine virtuelle.
- **Mise en réseau accélérée** : bien que cette fonctionnalité puisse être utile pour atteindre la limite publiée, elle ne modifie pas cette limite.
- **Destination du trafic** : toutes les destinations s’inscrivent dans la limite de sortie.
- **Protocole** : l’intégralité du trafic sortant, sur l’ensemble des protocoles, s’inscrit dans la limite.

## <a name="network-flow-limits"></a>Limites de flux de réseau

En plus de la bande passante, le nombre de connexions réseau présentes sur une machine virtuelle à un moment donné peut affecter ses performances réseau. La pile de mise en réseau Azure maintient l’état pour chaque direction d’une connexion TCP/UDP dans des structures de données appelées « flux ». Dans une connexion TCP/UDP par défaut, 2 flux seront créés, un pour la direction entrante et l’autre pour la direction sortante. 

Le transfert de données entre les points de terminaison nécessite la création de plusieurs flux en plus de ceux qui assurent le transfert de données. Certains exemples sont des flux créés pour une résolution DNS et des flux créés pour des sondes d’intégrité de l’équilibreur de charge. Notez également que les appliances réseau virtuelles (NVA) telles que les passerelles, les proxys et les pare-feu verront la création de flux pour des connexions arrêtées au niveau de l’appliance et en provenant. 

![Nombre de flux pour la conversation TCP via une appliance de transfert](media/virtual-machine-network-throughput/flow-count-through-network-virtual-appliance.png)

## <a name="flow-limits-and-recommendations"></a>Limites de flux et recommandations

Aujourd'hui, la pile de mise en réseau Azure prend en charge 250 000 flux de réseau au total, avec un bon niveau de performance pour les machines virtuelles ayant plus de 8 cœurs d’UC et 100 000 flux au total, avec un bon niveau de performance pour les machines virtuelles ayant moins de 8 cœurs d’UC. Au-delà de cette limite, le niveau de performance du réseau se dégrade normalement en cas de flux supplémentaires jusqu’à une limite inconditionnelle de 500 000 flux au total, à savoir respectivement 250 000 flux entrants et 250 000 flux sortants, après quoi les flux supplémentaires sont supprimés.

| Niveau de performance | Machines virtuelles avec < 8 cœurs d’UC | Machines virtuelles avec plus de 8 cœurs d’UC |
| ----------------- | --------------------- | --------------------- |
|<b>Bon niveau de performance</b>|100 000 flux |250 000 flux|
|<b>Performances dégradées</b>|Plus de 100 000 flux|Plus de 250 000 flux|
|<b>Limite de flux</b>|500 000 flux|500 000 flux|

Des métriques permettant de suivre le nombre de flux réseau et le taux de création des flux sur votre machine virtuelle ou sur des instances de VMSS sont disponibles dans [Azure Monitor](../azure-monitor/platform/metrics-supported.md#microsoftcomputevirtualmachines).

![Capture d’écran montrant la page Métriques d’Azure Monitor avec un graphique en courbes et des totaux pour les flux entrants et sortants.](media/virtual-machine-network-throughput/azure-monitor-flow-metrics.png)

Le taux d’établissement et de fin de connexions peut également affecter le niveau de performance réseau, car l’établissement et la fin des connexions partagent l’UC avec les routines de traitement de paquets. Nous vous recommandons d’évaluer les charges de travail pour les modèles de trafic attendus et d’effectuer un scale-out de manière appropriée en fonction de vos besoins en matière de niveau de performance. 

## <a name="next-steps"></a>Étapes suivantes

- [Optimiser le débit réseau d’un système d’exploitation de machine virtuelle](virtual-network-optimize-network-bandwidth.md)
- [Tester le débit réseau](virtual-network-bandwidth-testing.md) d’une machine virtuelle
