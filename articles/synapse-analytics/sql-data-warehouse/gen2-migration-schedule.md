---
title: Migrer votre pool SQL dédié (anciennement SQL DW) vers Gen2
description: Instructions concernant la migration d’un pool SQL dédié existant (anciennement SQL DW) vers Gen2 et la planification de la migration par région.
services: synapse-analytics
author: mlee3gsd
ms.author: anjangsh
ms.reviewer: jrasnick
manager: craigg
ms.assetid: 04b05dea-c066-44a0-9751-0774eb84c689
ms.service: synapse-analytics
ms.topic: article
ms.subservice: sql-dw
ms.date: 01/21/2020
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 512775369bd7787c6228c6d452be0e236ddf5cc2
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96456335"
---
# <a name="upgrade-your-dedicated-sql-pool-formerly-sql-dw-to-gen2"></a>Mettre à niveau votre pool SQL dédié (anciennement SQL DW) vers Gen2

Microsoft aide à réduire le coût d’entrée de gamme de l’exécution d’un pool SQL dédié (anciennement SQL DW).  Des niveaux de service de calcul plus bas capables de gérer des requêtes exigeantes sont désormais disponibles pour les pools SQL dédiés (anciennement SQL DW). Lire l’intégralité de l’annonce [Prise en charge du niveau de service de calcul plus bas Gen2](https://azure.microsoft.com/blog/azure-sql-data-warehouse-gen2-now-supports-lower-compute-tiers/). La nouvelle offre est disponible dans les régions indiquées dans le tableau ci-dessous. Pour les régions prises en charge, un pool SQL dédié Gen1 existant (anciennement SQL DW) peut être mis à niveau vers Gen2 à l’aide de l’une des méthodes suivantes :

- **Processus de mise à niveau automatique :** Les mises à niveau automatiques ne commencent pas dès que le service est disponible dans une région.  Quand les mises à niveau automatiques commenceront dans une région spécifique, les mises à niveau individuelles de pools SQL s’effectueront pendant la planification de maintenance que vous avez sélectionnée.
- [**Mise à niveau automatique vers Gen2 :**](#self-upgrade-to-gen2) Vous pouvez contrôler quand mettre à niveau en effectuant une mise à niveau automatique vers Gen2. Si votre région n’est pas encore prise en charge, vous pouvez restaurer à partir d’un point de restauration directement vers une instance Gen2 dans une région prise en charge.

## <a name="automated-schedule-and-region-availability-table"></a>Tableau des planifications automatisées et de la disponibilité dans les régions

Le tableau suivant récapitule par région le moment où le niveau de capacité de calcul Gen2 inférieur sera disponible et où les mises à niveau automatiques commencent. Les dates sont susceptibles d’être modifiées. Reconsultez-le pour voir quand votre région sera disponible.

\* indique qu’une planification spécifique pour la région est actuellement non disponible.

| **Région** | **Gen2 inférieur disponible** | **Commencement de la mise à niveau automatique** |
|:--- |:--- |:--- |
| Est du Canada |1er juin 2020 |1er juillet 2020 |
| Chine orientale |\* |\* |
| Chine du Nord |\* |\* |
| Centre de l’Allemagne |\* |\* |
| Allemagne Centre-Ouest |Disponible |1 mai 2020 |
| Inde Ouest |Disponible |1 mai 2020  |

## <a name="automatic-upgrade-process"></a>Processus de mise à niveau automatique

Selon le graphique de disponibilité ci-dessus, nous allons planifier des mises à niveau automatisées pour vos instances Gen1. Pour éviter des interruptions inattendues de la disponibilité du pool SQL dédié (anciennement SQL DW), les mises à niveau automatiques seront planifiées pendant votre planification de maintenance. La possibilité de créer une nouvelle instance Gen1 sera désactivée dans les régions où des mise à niveau automatiques vers Gen2 sont effectuées. Gen1 sera déconseillée une fois que les mises à niveau automatiques auront été effectuées. Pour plus d’informations sur les planifications, consultez [Afficher une planification de maintenance](maintenance-scheduling.md#view-a-maintenance-schedule).

Le processus de mise à niveau impliquera une brève baisse de connectivité (environ 5 min) quand nous redémarrons votre pool SQL dédié (anciennement SQL DW).  Une fois que votre pool SQL dédié (anciennement SQL DW) aura été redémarré, il sera entièrement disponible pour utilisation. Cependant, vous pouvez être confronté à une délai des performances pendant que le processus de mise à niveau continue à mettre à niveau les fichiers de données en arrière-plan. La durée totale de la dégradation des performances varie en fonction de la taille de vos fichiers de données.

Vous pouvez également accélérer le processus de mise à niveau des fichiers de données en exécutant la commande [Alter Index Rebuild](sql-data-warehouse-tables-index.md) sur toutes les tables columnstore primaires à l’aide d’une plus large classe de ressources et de SLO après le redémarrage.

> [!NOTE]
> Alter Index Rebuild est une opération hors connexion et les tables ne seront pas disponibles tant que la reconstruction ne sera pas terminée.

## <a name="self-upgrade-to-gen2"></a>Mise à niveau automatique vers Gen2

Vous pouvez choisir d’effectuer une mise à niveau automatique en suivant ces étapes sur un pool SQL dédié (anciennement SQL DW) Gen1 existant. Si vous choisissez la mise à niveau automatique, vous devez l’effectuer avant le début du processus de mise à niveau automatique dans votre région. Vous évitez ainsi tout risque de conflit dû aux mises à niveau automatique.

Deux options s’offrent à vous quand vous procédez à une mise à niveau automatique.  Vous pouvez soit mettre à niveau votre pool SQL dédié actuel (anciennement SQL DW) sur place, soit restaurer un pool SQL dédié Gen1 (anciennement SQL DW) dans une instance Gen2.

- [Mettre à niveau sur place](upgrade-to-latest-generation.md) : cette option met à niveau votre pool SQL dédié (anciennement SQL DW) Gen1 existant vers Gen2. Le processus de mise à niveau impliquera une brève baisse de connectivité (environ 5 min) quand nous redémarrons votre pool SQL dédié (anciennement SQL DW).  Une fois redémarré, il sera entièrement disponible pour utilisation. Si vous rencontrez des problèmes pendant la mise à niveau, ouvrez une [demande de support](sql-data-warehouse-get-started-create-support-ticket.md) et indiquez « Mise à niveau vers Gen2 » comme cause possible.
- [Mettre à niveau à partir du point de restauration](sql-data-warehouse-restore-points.md) : créez un point de restauration défini par l’utilisateur sur votre pool SQL dédié (anciennement SQL DW) Gen1 actuel, puis effectuez la restauration directement sur une instance Gen2. Le pool SQL dédié (anciennement SQL DW) Gen1 existant est conservé. Une fois la restauration terminée, votre pool SQL dédié (anciennement SQL DW) Gen2 sera entièrement disponible pour utilisation.  Une fois tous les processus de test et de validation exécutés sur l’instance Gen2 restaurée, l’instance Gen1 d’origine peut être supprimée.

  - Étape 1 : À partir du portail Azure, [créez un point de restauration défini par l’utilisateur](sql-data-warehouse-restore-active-paused-dw.md).
  - Étape 2 : Lors d’une restauration à partir d’un point de restauration défini par l’utilisateur, définissez le « niveau de performances » sur votre niveau Gen2 préféré.

Vous pouvez être confronté à un délai de dégradation des performances pendant que le processus de mise à niveau continue à mettre à niveau les fichiers de données en arrière-plan. La durée totale de la dégradation des performances varie en fonction de la taille de vos fichiers de données.

Pour accélérer le processus de migration de données en arrière-plan, vous pouvez forcer immédiatement le déplacement des données en exécutant la commande [Alter Index Rebuild](sql-data-warehouse-tables-index.md) sur toutes les tables columnstore primaires que vous interrogez sur une plus large classe de SLO et de ressources.

> [!NOTE]
> Alter Index Rebuild est une opération hors connexion et les tables ne seront pas disponibles tant que la reconstruction ne sera pas terminée.

Si vous rencontrez des problèmes avec votre pool SQL dédié (anciennement SQL DW), créez une [demande de support](sql-data-warehouse-get-started-create-support-ticket.md) et indiquez « Mise à niveau vers Gen2 » comme cause possible.

Pour plus d’informations, consultez [Mettre à niveau vers Gen2](upgrade-to-latest-generation.md).

## <a name="migration-frequently-asked-questions"></a>Questions fréquentes sur la migration

**Q : Le coût de Gen2 est-il le même que celui de Gen1 ?**

- A : Oui.

**Q : Comment les mises à niveau affecteront-elles mes scripts d’automatisation ?**

- A : Tout script d’automatisation qui fait référence à un objectif de niveau de service doit être changé pour correspondre à l’équivalent Gen2.  Consultez les informations détaillées [ici](upgrade-to-latest-generation.md#upgrade-in-a-supported-region-using-the-azure-portal).

**Q : Combien de temps prend normalement une mise à niveau automatique ?**

- A : Vous pouvez effectuer une mise à niveau sur place ou à partir d’un point de restauration.

  - Une mise à niveau sur place provoque une suspension momentanée puis une reprise de votre pool SQL dédié (anciennement SQL DW).  Un processus en arrière-plan continue pendant que le pool SQL dédié (anciennement SQL DW) est en ligne.  
  - L’opération prend plus de temps si vous effectuez la mise à niveau par le biais d’un point de restauration, car la mise à niveau passera par le processus de restauration complète.

**Q : Combien de temps prend la mise à niveau automatique ?**

- A : Le temps d’arrêt réel pour la mise à niveau est uniquement le temps nécessaire pour suspendre et reprendre le service, qui est compris entre 5 et 10 minutes. Après la courte interruption, un processus en arrière-plan exécutera une migration de stockage. La durée du processus en arrière-plan dépend de la taille de votre pool SQL dédié (anciennement SQL DW).

**Q : Quand cette mise à niveau automatique aura-t-elle lieu ?**

- A : Pendant votre planification de maintenance. Exploiter la planification de maintenance que vous avez choisie limite les perturbations pour votre activité.

**Q : Que dois-je faire si mon processus de mise à niveau en arrière-plan semble être bloqué ?**

- A : Lancez une réindexation de vos tables Columnstore. Notez que la réindexation de la table s’effectuera hors connexion.

**Q : Que se passe-t-il si Gen2 n’a pas l’objectif de niveau de service que j’ai sur Gen1 ?**

- A : Si vous exécutez DW600 ou DW1200 sur Gen1, il est conseillé d’utiliser DW500c ou DW1000c, respectivement, car Gen2 fournit plus de mémoire, plus de ressources et des performances plus élevées que Gen1.

**Q : Puis-je désactiver la géosauvegarde ?**

- A : Non. La géosauvegarde est une fonctionnalité d’entreprise permettant de préserver la disponibilité de votre pool SQL dédié (anciennement SQL DW) dans le cas où une région n’est plus disponible. Si vous avez d’autres questions, ouvrez une [demande de support](sql-data-warehouse-get-started-create-support-ticket.md).

**Q : Existe-t-il une différence de syntaxe T-SQL entre Gen1 et Gen2 ?**

- A : Aucun changement n’affecte la syntaxe du langage T-SQL entre Gen1 et Gen2.

**Q : Gen2 prend-il en charge les fenêtres de maintenance ?**

- A : Oui.

**Q : Pourrai-je créer une instance Gen1 après la mise à niveau de ma région ?**

- A : Non. Une fois qu’une région aura été mise à niveau, la création d’instances Gen1 sera désactivée.

## <a name="next-steps"></a>Étapes suivantes

- [Étapes de la mise à niveau](upgrade-to-latest-generation.md)
- [Fenêtres de maintenance](maintenance-scheduling.md)
- [Supervision de l’intégrité des ressources](../../service-health/resource-health-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
- [Revue avant de commencer une migration](upgrade-to-latest-generation.md#before-you-begin)
- [Mise à niveau sur place et mise à niveau à partir d’un point de restauration](upgrade-to-latest-generation.md)
- [Créer un point de restauration défini par l’utilisateur](sql-data-warehouse-restore-points.md)
- [Découvrir comment effectuer une restauration vers Gen2](sql-data-warehouse-restore-active-paused-dw.md)
- [Ouvrir une demande de support Azure Synapse Analytics](https://go.microsoft.com/fwlink/?linkid=857950)
