---
title: Sélectionner une option de migration VMware avec la migration de serveur Azure Migrate
description: Fournit une vue d’ensemble des options de migration de machines virtuelles VMware vers Azure avec la migration de serveur Azure Migrate
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: 5e4aaea5b565e126f633b04215bbc3a24faf2cde
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753601"
---
# <a name="select-a-vmware-migration-option"></a>Sélectionnez une option de migration VMware

Vous pouvez migrer des machines virtuelles VMware vers Azure à l’aide de l’outil de migration de serveur Azure Migrate. Cet outil offre deux options pour la migration de machines virtuelles VMware :

- Migration à l’aide de la réplication sans agent. Migrez des machines virtuelles sans avoir à installer quoi que ce soit.
- Migration avec un agent pour la réplication. Installez un agent sur la machine virtuelle pour la réplication.


## <a name="compare-migration-methods"></a>Comparer les méthodes de migration

Utilisez ces comparaisons sélectionnées pour vous aider à choisir la méthode à utiliser. Vous pouvez également consulter les spécifications de prise en charge complète pour la migration [sans agent](migrate-support-matrix-vmware-migration.md#agentless-migration) et [par agent](migrate-support-matrix-vmware-migration.md#agent-based-migration).

**Paramètre** | **Sans agent** | **Basé sur un agent**
--- | --- | ---
**Autorisations Azure** | Vous avez besoin d’autorisations pour créer un projet Azure Migrate et pour inscrire des applications Azure AD créées lorsque vous déployez l’appliance Azure Migrate. | Vous avez besoin des autorisations de Collaborateur sur l’abonnement Azure. 
**Réplication** | Un maximum de 300 machines virtuelles peuvent être répliquées simultanément à partir d’un vCenter Server.<br/> Si vous avez plus de 50 machines virtuelles pour la migration, créez plusieurs lots de machines virtuelles.<br/> La réplication multiple en une seule fois aura un impact sur les performances.<br/><br/> Dans le portail, vous pouvez sélectionner jusqu’à 10 machines à la fois pour la réplication. Pour répliquer davantage de machines, ajoutez-les par lots de 10.| La capacité de réplication augmente en mettant à l’échelle l’appliance de réplication.
**Étapes de déploiement d’appliance** | L'[appliance Azure Migrate](migrate-appliance.md) est déployée en local. | L'[appliance de réplication Azure Migrate](migrate-replication-appliance.md) est déployée en local.
**Compatibilité Site Recovery** | Compatible. | Vous ne pouvez pas répliquer avec Azure Migrate Server Migration si vous avez défini la réplication pour une machine utilisant Site Recovery.
**Disque cible** | Disques managés | Disques managés
**Limites du disque** | Disque de système d’exploitation : 2 To<br/><br/> Disque de données : 32 To<br/><br/> Nombre maximal de disques : 60 | Disque de système d’exploitation : 2 To<br/><br/> Disque de données : 8 To<br/><br/> Nombre maximal de disques : 63
**Disques directs** | Non pris en charge | Prise en charge
**Démarrage UEFI** | Pris en charge. | Pris en charge.

## <a name="compare-deployment-steps"></a>Comparer les étapes de déploiement

Après avoir examiné les limitations, la compréhension des étapes impliquées dans le déploiement de chaque solution peut vous aider à choisir l’option appropriée.

**Tâche** | **Détails** |**Sans agent** | **Basé sur un agent**
--- | --- | --- | ---
**Déployez l’appliance Azure Migrate** | Une appliance légère qui s’exécute sur une machine virtuelle VMware.<br/><br/> L’appliance est utilisée pour détecter et évaluer les machines, et pour migrer des machines à l’aide d’une migration sans agent. | Obligatoire.<br/><br/> Si vous avez déjà configuré l’appliance pour l’évaluation, vous pouvez utiliser la même appliance pour la migration sans agent. | Non requis.<br/><br/> Si vous avez configuré une appliance à des fins d’évaluation, vous pouvez la conserver ou la supprimer si vous avez terminé l’évaluation.
**Utiliser l’outil d’évaluation de serveur** | Évaluez des machines virtuelles VMware avec l’outil Azure Migrate : Server Assessment. | Vous pouvez évaluer les machines avant de les migrer, mais ce n’est pas une obligation. | L’évaluation est facultative | L’évaluation est facultative.
**Utiliser l’outil Server Migration** | Ajoutez l’outil de migration de serveur Azure Migrate dans le projet Azure Migrate. | Obligatoire | Obligatoire
**Préparer VMware pour la migration** | Configurez les paramètres sur les serveurs VMware et les machines virtuelles. | Obligatoire | Obligatoire
**Installez le service Mobilité sur des machines virtuelles** | Le service Mobilité s’exécute sur chaque machine virtuelle que vous souhaitez répliquer | Non requis | Obligatoire
**Déployer l’appliance de réplication** | [L’appliance de réplication](migrate-replication-appliance.md) est utilisée pour la migration basée sur les agents. Il se connecte entre le service Mobilité en cours d’exécution sur les machines virtuelles et Server Migration. | Non requis | Obligatoire
**Répliquez les machines virtuelles**. Activez la réplication de machines virtuelles. | Configurez les paramètres de réplication et sélectionner les machines virtuelles à répliquer | Obligatoire | Obligatoire
**Exécutez un test de migration** | Exécuter une migration de test pour vérifier que tout fonctionne comme prévu. | Obligatoire | Obligatoire
**Exécutez une migration complète** | Migrez les machines virtuelles. | Obligatoire | Obligatoire



## <a name="next-steps"></a>Étapes suivantes

[Migrez des machines virtuelles VMware](tutorial-migrate-vmware.md) avec une migration sans agent.



