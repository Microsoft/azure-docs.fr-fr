---
title: Activer la mise en réseau accélérée pour la récupération d’urgence de machines virtuelles Azure avec Azure Site Recovery
description: Explique comment utiliser la mise en réseau accélérée avec Azure Site Recovery pour la récupération d’urgence de machines virtuelles Azure
services: site-recovery
documentationcenter: ''
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: harshacs
ms.openlocfilehash: 1d2d3b3aacc00428c96cde0f8230421a98151ae2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000247"
---
# <a name="accelerated-networking-with-azure-virtual-machine-disaster-recovery"></a>Mise en réseau accélérée avec la récupération d’urgence de machines virtuelles Azure

Une mise en réseau accélérée permet d’opérer une virtualisation d’E/S d’une racine unique (SR-IOV) sur une machine virtuelle, ce qui améliore considérablement les performances de mise en réseau. Cette voie hautement performante court-circuite l’hôte à partir du chemin d’accès aux données, réduisant ainsi la latence, l’instabilité et l’utilisation du processeur pour servir les charges de travail réseau les plus exigeantes sur les types de machines virtuelles pris en charge. L’illustration suivante montre la communication entre deux machines virtuelles avec ou sans mise en réseau accélérée :

![Comparaison](./media/azure-vm-disaster-recovery-with-accelerated-networking/accelerated-networking-benefit.png)

Azure Site Recovery vous permet d’utiliser les avantages de la mise en réseau accélérée, pour les machines virtuelles Azure basculées vers une autre région Azure. Cet article décrit comment utiliser la mise en réseau accélérée pour les machines virtuelles Azure répliquées avec Azure Site Recovery.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous connaissez suffisamment les sujets suivants :
-   [Architecture de réplication](azure-to-azure-architecture.md) des machines virtuelles Azure
-   [Configuration de la réplication](azure-to-azure-tutorial-enable-replication.md) pour les machines virtuelles
-   [Basculement](azure-to-azure-tutorial-failover-failback.md) des machines virtuelles Azure

## <a name="accelerated-networking-with-windows-vms"></a>Mise en réseau accélérée avec des machines virtuelles Windows

Azure Site Recovery prend en charge l’activation de la mise en réseau accélérée pour les machines virtuelles répliquées seulement si celle-ci est activée pour la machine virtuelle source. Si la mise en réseau accélérée n’est pas activée pour votre machine virtuelle source, vous pouvez apprendre [ici](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms) à activer la mise en réseau accélérée pour les machines virtuelles Windows.

### <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge
Les distributions suivantes sont prises en charge sans configuration supplémentaire à partir de la galerie Azure :
* **Windows Server 2016 Datacenter**
* **Windows Server 2012 R2 Datacenter**

### <a name="supported-vm-instances"></a>Instances de machines virtuelles prises en charge
La mise en réseau accélérée est prise en charge dans la plupart des instances d’usage général et optimisées pour le calcul (2 processeurs virtuels ou plus).  Séries prises en charge : D/DSv2 et F/Fs

Dans des instances qui acceptent l’hyperthreading, la mise en réseau accélérée est prise en charge dans des instances de machine virtuelle comptant au minimum 4 processeurs virtuels. Séries prises en charge : D/DSv3, E/ESv3, Fsv2 et Ms/Mms

Pour plus d’informations sur les instances de machine virtuelle, consultez la section [Tailles des machines virtuelles Windows](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="accelerated-networking-with-linux-vms"></a>Mise en réseau accélérée avec des machines virtuelles Linux

Azure Site Recovery prend en charge l’activation de la mise en réseau accélérée pour les machines virtuelles répliquées seulement si celle-ci est activée pour la machine virtuelle source. Si la mise en réseau accélérée n’est pas activée pour votre machine virtuelle source, vous pouvez apprendre [ici](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms) à activer la mise en réseau accélérée pour les machines virtuelles Linux.

### <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge
Les distributions suivantes sont prises en charge sans configuration supplémentaire à partir de la galerie Azure :
* **Ubuntu 16.04**
* **SLES 12 SP3**
* **RHEL 7.4**
* **CentOS 7.4**
* **CoreOS Linux**
* **Debian « Stretch » with backports kernel**
* **Oracle Linux 7.4**

### <a name="supported-vm-instances"></a>Instances de machines virtuelles prises en charge
La mise en réseau accélérée est prise en charge dans la plupart des instances d’usage général et optimisées pour le calcul (2 processeurs virtuels ou plus).  Séries prises en charge : D/DSv2 et F/Fs

Dans des instances qui acceptent l’hyperthreading, la mise en réseau accélérée est prise en charge dans des instances de machine virtuelle comptant au minimum 4 processeurs virtuels. Séries prises en charge : D/DSv3, E/ESv3, Fsv2 et Ms/Mms.

Pour plus d’informations sur les instances de machine virtuelle, consultez la section [Tailles des machines virtuelles Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="enabling-accelerated-networking-for-replicated-vms"></a>Activation de la mise en réseau accélérée sur des machines virtuelles répliquées

Lorsque vous [activez la réplication](azure-to-azure-tutorial-enable-replication.md) pour des machines virtuelles Azure, Site Recovery détecte automatiquement si la mise en réseau accélérée est activée pour les interfaces réseau de machine virtuelle. Si la mise en réseau accélérée est déjà activée, Site Recovery configurera automatiquement la mise en réseau accélérée sur les interfaces réseau de la machine virtuelle répliquée.

L’état de la mise en réseau accélérée peut être vérifié dans la section **Interfaces réseau** des paramètres **Calcul et réseau** pour la machine virtuelle répliquée.

![Paramètre de la mise en réseau accélérée](./media/azure-vm-disaster-recovery-with-accelerated-networking/compute-network-accelerated-networking.png)

Si vous avez activé la mise en réseau accélérée sur la machine virtuelle source après avoir activé la réplication, vous pouvez activer la mise en réseau accélérée pour les interfaces réseau de la machine virtuelle répliquée avec le processus suivant :
1. Ouvrez les paramètres **Calcul et réseau** pour la machine virtuelle répliquée
2. Cliquez sur le nom de l’interface réseau sous la section **Interface réseau**
3. Sélectionnez **Activé** dans la liste déroulante pour la mise en réseau accélérée sous la colonne **Cible**

![Activer la mise en réseau accélérée](./media/azure-vm-disaster-recovery-with-accelerated-networking/network-interface-accelerated-networking-enabled.png)

Le processus ci-dessus doit également être suivi pour les machines virtuelles répliquées existantes, qui n’avaient pas la mise en réseau accélérée précédemment activée automatiquement par Site Recovery.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [avantages de la mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-powershell.md#benefits).
- En savoir plus sur les limitations et les contraintes de mise en réseau accélérée pour des [machines virtuelles Windows](../virtual-network/create-vm-accelerated-networking-powershell.md#limitations-and-constraints) et des [machines virtuelles Linux](../virtual-network/create-vm-accelerated-networking-cli.md#limitations-and-constraints).
- En savoir plus sur les [plans de récupération](site-recovery-create-recovery-plans.md) pour automatiser le basculement d’application