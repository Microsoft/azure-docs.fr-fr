---
title: Migrer des machines locales avec Azure Migrate
description: Cet article résume comment migrer des machines locales vers Azure et recommande Azure Migrate.
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/27/2020
ms.author: raynew
ms.openlocfilehash: e0e60ee346d20113b2ec7970390d9874522cc770
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87847310"
---
# <a name="migrate-on-premises-machines-to-azure"></a>Migrer des machines sur site vers Azure

Cet article décrit les options de migration des machines locales vers Azure. 

## <a name="migrate-with-azure-migrate"></a>Migrer avec Azure Migrate

Nous vous recommandons de migrer des machines vers Azure à l’aide du service [Azure Migrate](../migrate/migrate-services-overview.md). Azure Migrate est spécialement conçu pour la migration de serveur. Azure Migrate fournit un hub centralisé pour la découverte, l’évaluation et la migration des machines locales vers Azure.

Pour migrer avec Azure Migrate, cliquez sur les liens suivants :

- [En savoir plus sur](../migrate/server-migrate-overview.md) les options de migration pour les machines virtuelles VMware, puis la migration des machines virtuelles vers Azure à l’aide d’une migration [sans agent](../migrate/tutorial-migrate-vmware.md) ou [avec agent](../migrate/tutorial-migrate-vmware-agent.md).
- [Migrer des machines virtuelles Hyper-V](../migrate/tutorial-migrate-hyper-v.md) vers Azure.
- Migrez des [serveurs physiques ou d’autres machines virtuelles](../migrate/tutorial-migrate-physical-virtual-machines.md), notamment des instances [AWS](../migrate/tutorial-migrate-aws-virtual-machines.md) vers Azure.

## <a name="migrate-with-site-recovery"></a>Migrer avec Site Recovery
Site Recovery ne doit être utilisé que pour la récupération d’urgence, pas pour la migration.

Si vous utilisez déjà Azure Site Recovery et souhaitez continuer à l’utiliser pour la migration, procédez comme pour configurer la récupération d’urgence.

- Machines virtuelles VMware : [Préparez Azure](tutorial-prepare-azure.md) et [VMware](vmware-azure-tutorial-prepare-on-premises.md), commencez la [réplication des machines](vmware-azure-tutorial.md), [vérifiez](tutorial-dr-drill-azure.md) que tout fonctionne et [exécutez un basculement](vmware-azure-tutorial-failover-failback.md).
- Machines virtuelles Hyper-V : [Préparez Azure](tutorial-prepare-azure-for-hyperv.md) et [Hyper-V](hyper-v-prepare-on-premises-tutorial.md), commencez la [réplication des machines](hyper-v-azure-tutorial.md), [vérifiez](tutorial-dr-drill-azure.md) que tout fonctionne et [exécutez un basculement](hyper-v-azure-failover-failback-tutorial.md).
- Serveurs physiques : [Suivez la procédure pas à pas](physical-azure-disaster-recovery.md) pour préparer Azure, préparer des machines pour la récupération d’urgence et configurer la réplication.

> [!NOTE]
> Lorsque vous opérez un basculement à des fins de récupération d’urgence, lors de la dernière étape, vous validez le basculement. Lorsque vous migrez des machines locales, l’option **Valider** n’est pas pertinente. Sélectionnez à la place l’option **Terminer la migration**. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Passez en revue les questions courantes](../migrate/resources-faq.md) sur Azure Migrate.

  
