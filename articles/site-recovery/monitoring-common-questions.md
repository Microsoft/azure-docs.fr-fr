---
title: Questions courantes sur la supervision Azure Site Recovery
description: Obtenez des réponses aux questions courantes sur la supervision Azure Site Recovery, effectuée à l’aide de la supervision intégrée et d’Azure Monitor (Log Analytics)
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
services: site-recovery
ms.date: 07/31/2019
ms.topic: conceptual
ms.author: raynew
ms.openlocfilehash: 66ce267581d4748ea51a3dcbd7caa61907115cc1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82131161"
---
# <a name="common-questions-about-site-recovery-monitoring"></a>Questions fréquentes sur la supervision Site Recovery

Cet article répond aux questions fréquemment posées concernant la supervision Azure [Site Recovery](site-recovery-overview.md), effectuée à l’aide de la supervision intégrée Site Recovery et d’Azure Monitor (Log Analytics).

## <a name="general"></a>Général

### <a name="how-is-the-rpo-value-logged-different-from-the-latest-available-recovery-point"></a>En quoi le RPO est-il différent du dernier point de récupération disponible ?

Site Recovery utilise un processus asynchrone en plusieurs étapes pour répliquer des machines dans Azure.

- À l’avant-dernière étape de la réplication, les modifications récentes apportées à la machine, ainsi que les métadonnées, sont copiées dans un compte de stockage de cache/du journal.
- Ces modifications, ainsi que l’étiquette qui identifie un point récupérable, sont écrites dans le compte de stockage ou sur le disque managé de la région cible.
- Site Recovery peut désormais générer un point récupérable pour la machine.
- À ce stade, le RPO a été atteint pour les modifications chargées jusqu’à présent dans le compte de stockage. En d’autres termes, le RPO de la machine est à ce stade égal au temps écoulé depuis l’horodatage correspondant au point récupérable.
- À présent, Site Recovery récupère les données téléchargées à partir du compte de stockage et les applique aux disques de réplica créés pour la machine.
- Site Recovery génère ensuite un point de récupération et le fournit pour effectuer la récupération lors du basculement.
- Le dernier point de récupération disponible indique l’horodatage correspondant au dernier point de récupération déjà traité et appliqué aux disques de réplica.


Une heure système incorrecte sur la machine source en cours de réplication ou les serveurs d’infrastructure locaux fausse la valeur de RPO calculée. Pour obtenir un RPO précis, assurez-vous que l’horloge système affiche la même heure sur l’ensemble des serveurs et des machines.



## <a name="inbuilt-site-recovery-logging"></a>Journalisation Site Recovery intégrée


### <a name="why-is-the-vm-count-in-the-vault-infrastructure-view-different-from-the-total-count-shown-in-replicated-items"></a>Pourquoi le nombre de machines virtuelles dans la vue d’infrastructure du coffre diffère-t-il du nombre total affiché dans la section des éléments répliqués ?

La vue d’infrastructure du coffre est limitée par les scénarios de réplication. Seules les machines incluses dans le scénario de réplication sélectionné sont comptabilisées dans la vue d’infrastructure. En outre, nous comptabilisons uniquement les machines virtuelles qui sont configurées pour être répliquées vers Azure. Les machines ayant basculé ou en cours de réplication vers un site local ne sont pas comptabilisées dans la vue.

### <a name="why-is-the-count-of-replicated-items-in-essentials-different-from-the-total-count-of-replicated-items-on-the-dashboard"></a>Pourquoi le nombre d’éléments répliqués indiqué dans la section des éléments fondamentaux diffère du nombre total d’éléments répliqués indiqué dans le tableau de bord ?

Seules les machines sur lesquelles la réplication initiale est terminée sont incluses dans le nombre indiqué dans la section des éléments fondamentaux. Dans la section des éléments répliqués, le total inclut toutes les machines présentes dans le coffre, y compris celles dont la réplication initiale est en cours.

## <a name="azure-monitor-logging"></a>Journalisation d’Azure Monitor


### <a name="how-often-does-site-recovery-send-resource-logs-to-azure-monitor-log"></a>À quelle fréquence Site Recovery envoie-t-il des journaux de ressources aux journaux d’activité Azure Monitor ? 

- Le journal des statistiques de réplication (AzureSiteRecoveryReplicationStats) et le journal des points de récupération (AzureSiteRecoveryRecoveryPoints) sont envoyés toutes les 15 minutes.  
- Le journal des taux de chargement des données de réplication (AzureSiteRecoveryReplicationDataUploadRate) et le journal de l’évolution des données des disques protégés (AzureSiteRecoveryProtectedDiskDataChurn) sont envoyés toutes les 5 minutes. 
- Le journal des travaux de récupération (AzureSiteRecoveryJobs) est envoyé lors du déclenchement et de l’achèvement d’un travail.
- Le journal des événements de récupération (AzureSiteRecoveryEvents) est envoyé chaque fois qu’un événement est généré. 
- Le journal des éléments répliqués (AzureSiteRecoveryReplicatedItems) est envoyé à chaque modification de l’environnement. En règle générale, le temps d’actualisation des données est de 15 minutes après une modification. 

### <a name="how-long-is-data-kept-in-azure-monitor-logs"></a>Combien de temps les données sont conservées dans les journaux Azure Monitor ? 

Par défaut, la période de conservation est de 31 jours. Vous pouvez augmenter cette durée dans la section **Coûts d’utilisation et estimations** de l’espace de travail Log Analytics. Cliquez sur **Conservation des données**, puis sélectionnez une plage.

### <a name="whats-the-size-of-the-resource-logs"></a>Quelle est la taille d’un journal de ressources ? 

En règle générale, la taille d’un journal est de 15 à 20 Ko. 


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment superviser avec la [supervision intégrée Site Recovery](site-recovery-monitor-and-troubleshoot.md) ou [Azure Monitor](monitor-log-analytics.md).


