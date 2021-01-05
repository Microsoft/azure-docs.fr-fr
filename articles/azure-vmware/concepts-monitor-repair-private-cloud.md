---
title: Concepts – Surveiller et réparer des clouds privés Azure VMware Solution
description: Découvrez comment Azure VMware Solution surveille et répare les serveurs VMware ESXi sur un cloud privé Azure VMware Solution.
ms.topic: conceptual
ms.custom: contperf-fy21q2
ms.date: 11/20/2020
ms.openlocfilehash: 7f367a6ef9b86f235a16a698735cfb58a11f6862
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033593"
---
# <a name="monitor-and-repair-azure-vmware-solution-private-clouds"></a>Surveiller et réparer des clouds privés Azure VMware Solution

Azure VMware Solution surveille continuellement les serveurs VMware ESXi sur un cloud privé Azure VMware Solution. 

## <a name="what-azure-vmware-solution-monitors"></a>Éléments surveillés par Azure VMware Solution

Azure VMware Solution surveille les éléments suivants pour détecter des conditions d’échec sur l’hôte :  

- État des processeurs 
- État de la mémoire 
- État des connexions et de l’alimentation 
- État du ventilateur matériel 
- Perte de connectivité réseau 
- État de la carte système matérielle 
- Des erreurs se sont produites sur le ou les disques d’un hôte vSAN 
- Tension du matériel 
- État de la température du matériel 
- État de l’alimentation du matériel 
- État du stockage 
- Échec de connexion 

> [!NOTE]
> Les administrateurs de locataires Azure VMware Solution ne doivent ni modifier ni supprimer les alarmes VMware vCenter définies ci-dessus, car celles-ci sont gérées par le plan de contrôle Azure VMware Solution sur vCenter. Ces alarmes sont utilisées par l’analyse de la solution VMware Azure pour déclencher le processus de correction de l’hôte Azure VMware Solution.

## <a name="azure-vmware-solution-host-remediation"></a>Correction de l’hôte Azure VMware Solution  

Quand Azure VMware Solution détecte une dégradation ou une défaillance sur un nœud Azure VMware Solution sur le cloud privé d’un locataire, un processus de correction de l’hôte est déclenché. La correction de l’hôte implique le remplacement du nœud défaillant par un nouveau nœud sain.  

Le processus de correction de l’hôte commence par l’ajout d’un nouveau nœud sain dans le cluster. Ensuite, lorsque cela est possible, l’hôte défectueux est placé en mode de maintenance VMware vSphere. VMware vMotion est utilisé pour déplacer les machines virtuelles de l’hôte défaillant vers d’autres serveurs disponibles dans le cluster, permettant ainsi éventuellement une migration dynamique des charges de travail sans temps d’arrêt. Dans les scénarios où l’hôte défaillant ne peut pas être placé en mode de maintenance, l’hôte est supprimé du cluster.

## <a name="next-steps"></a>Étapes suivantes

Voici quelques rubriques pour en savoir plus :

- [Mises à niveau de clouds privés Azure VMware Solution](concepts-upgrades.md)
- [Gestion du cycle de vie des machines virtuelles du service Azure VMware Solution](lifecycle-management-of-azure-vmware-solution-vms.md)
- [Protégez vos machines virtuelles Azure VMware Solution avec l’intégration Azure Security Center](azure-security-integration.md)
- [Sauvegarder des machines virtuelles Azure VMware Solution avec le serveur de sauvegarde Azure](backup-azure-vmware-solution-virtual-machines.md)
- [Effectuer une récupération d'urgence de machines virtuelles à l'aide d’Azure VMware Solution](disaster-recovery-for-virtual-machines.md)
