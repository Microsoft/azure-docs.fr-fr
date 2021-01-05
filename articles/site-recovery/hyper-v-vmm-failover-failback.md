---
title: Configurer le basculement et la restauration automatique vers un site Hyper-V secondaire avec Azure Site Recovery
description: Découvrez comment basculer des machines virtuelles Hyper-V vers votre site local secondaire et restaurer sur le site principal pendant une récupération d’urgence avec Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: d31355bcb0ce42874c19988738ba06138c7a0b7c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74082603"
---
# <a name="fail-over-and-fail-back-hyper-v-vms-replicated-to-your-secondary-on-premises-site"></a>Basculer et restaurer automatiquement des machines virtuelles Hyper-V répliquées sur votre site local secondaire

Le service [Azure Site Recovery](site-recovery-overview.md) gère et orchestre la réplication, le basculement et la restauration automatique des machines locales et des machines virtuelles Azure.

Cet article explique comment basculer une machine virtuelle Hyper-V managée dans un cloud System Center Virtual Machine Manager (VMM) vers un site VMM secondaire. Une fois que vous avez procédé au basculement, vous restaurez automatiquement sur votre site local quand il est disponible. Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Basculer une machine virtuelle Hyper-V d’un cloud VMM principal vers un cloud VMM secondaire
> * Reprotéger et effectuer une restauration automatique du site secondaire sur le site principal
> * Effectuer de nouveau une réplication du site principal sur le site secondaire (facultatif)

## <a name="failover-and-failback"></a>Basculement et restauration automatique

Le basculement et la restauration automatique comportent trois étapes :

1. **Basculement vers le site secondaire** : les machines basculent du site principal vers le site secondaire.
2. **Restauration automatique à partir du site secondaire** : les machines virtuelles sont répliquées depuis le site secondaire sur le site principal, et un basculement planifié est exécuté pour la restauration automatique.
3. Après le basculement planifié, vous pouvez, si vous le souhaitez, effectuer de nouveau une réplication du site primaire sur le site secondaire.


## <a name="prerequisites"></a>Prérequis

- Veillez à effectuer un [test de récupération d’urgence](hyper-v-vmm-test-failover.md) pour vérifier que tout fonctionne comme prévu.
- Pour effectuer la restauration automatique, vérifiez que les serveurs VMM principal et secondaire sont connectés à Site Recovery.



## <a name="run-a-failover-from-primary-to-secondary"></a>Effectuer un basculement du site principal vers le site secondaire

Vous pouvez effectuer un basculement régulier ou planifié pour des machines virtuelles Hyper-V.

- Utilisez un basculement régulier pour les interruptions inattendues. Quand vous effectuez ce basculement, Site Recovery crée une machine virtuelle sur le site secondaire et la démarre. Une perte de données peut se produire en fonction des données en attente qui n’ont pas été synchronisées.
- Un basculement planifié peut être utilisé pour la maintenance ou lors d’une interruption prévue. Cette option permet de ne perdre aucune donnée. Quand un basculement planifié est déclenché, les machines virtuelles sources sont arrêtées. Les données non synchronisées sont synchronisées et le basculement est déclenché. 
- 
  Cette procédure explique comment effectuer régulièrement un basculement.


1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine virtuelle > **Basculer**.
1. Sélectionnez **Arrêter la machine avant de commencer le basculement** si vous souhaitez que Site Recovery tente d’arrêter les machines virtuelles sources avant de déclencher le basculement. Site Recovery tente également de synchroniser les données locales qui n’ont pas encore été envoyées au site secondaire avant de déclencher le basculement. Notez que le basculement continue même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
2. La machine virtuelle devrait à présent apparaître dans le cloud VMM secondaire.
3. Après avoir vérifié la machine virtuelle, **validez** le basculement. Cela supprime tous les points de récupération disponibles.

> [!WARNING]
> **N’annulez pas un basculement en cours** : Avant le démarrage du basculement, la réplication de la machine virtuelle est arrêtée. Si vous annulez un basculement en cours, le basculement s’arrête mais la machine virtuelle ne sera pas à nouveau répliquée.  


## <a name="reverse-replicate-and-failover"></a>Réplication inverse et basculement

Démarrez la réplication du site secondaire sur le site principal et effectuez la restauration automatique sur le site principal. Quand les machines virtuelles fonctionnent de nouveau sur le site principal, vous pouvez les répliquer sur le site secondaire.  

 
1. Cliquez sur la machine virtuelle > Cliquez sur **Réplication inverse**.
2. Une fois la tâche terminée, cliquez sur la machine virtuelle dans **Basculement**, vérifiez le sens du basculement (à partir du cloud VMM secondaire) et sélectionnez les emplacements source et cible. 
4. Lancez le basculement. Vous pouvez suivre la progression du basculement sur l’onglet **Tâches** .
5. Dans le cloud VMM principal, vérifiez que la machine virtuelle est disponible.
6. Si vous souhaitez répliquer de nouveau la machine virtuelle principale sur le site secondaire, cliquez sur **Réplication inverse**.

## <a name="next-steps"></a>Étapes suivantes
[Passez en revue l’étape](hyper-v-vmm-disaster-recovery.md) sur la réplication des machines virtuelles Hyper-V vers un site secondaire.
