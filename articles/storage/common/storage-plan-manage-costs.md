---
title: Planifier et gérer les coûts du stockage Blob Azure
description: Découvrez comment planifier et gérer les coûts du stockage Blob Azure avec l’analyse du coût dans le portail Azure.
services: storage
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 11/13/2020
ms.author: normesta
ms.subservice: common
ms.custom: subject-cost-optimization
ms.openlocfilehash: 6942c3ecd631a141f1df741ff827a6889605e536
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498198"
---
# <a name="plan-and-manage-costs-for-azure-blob-storage"></a>Planifier et gérer les coûts du stockage Blob Azure

Cet article explique comment planifier et gérer les coûts du stockage Blob Azure. Tout d’abord, utilisez la calculatrice de prix Azure pour estimer les coûts. Après avoir créé votre compte de stockage, optimisez le compte afin de payer uniquement ce dont vous avez besoin. Utilisez les fonctionnalités de gestion des coûts pour définir des budgets et surveiller les coûts. Vous pouvez également passer en revue les coûts prévus et surveiller les tendances des dépenses pour identifier les domaines où vous pourriez agir.

Gardez à l’esprit que les coûts du stockage Blob ne représentent qu’une partie des coûts mensuels sur votre facture Azure. Cet article explique comment estimer et gérer les coûts du stockage Blob. Cependant, vous êtes facturé pour tous les services et ressources Azure utilisés pour votre abonnement Azure, y compris les services tiers. Quand vous êtes familiarisé avec la gestion des coûts du stockage Blob, vous pouvez appliquer des méthodes similaires pour gérer les coûts de tous les services Azure utilisés dans votre abonnement.

## <a name="estimate-costs"></a>Estimer les coûts

Utilisez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/) pour estimer les coûts avant de créer un compte de stockage Azure et de commencer à transférer des données vers ce compte.

1. Sur la page [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/), choisissez la vignette **Comptes de stockage**.

2. Faites défiler la page vers le bas et recherchez la section **Comptes de stockage** de votre estimation.

3. Choisissez les options souhaitées dans les listes déroulantes. 

   L’estimation du coût est mise à jour à mesure que vous modifiez les valeurs de ces listes déroulantes. Cette estimation s’affiche dans le coin supérieur et en bas de l’écran d’estimation.

   ![Capture d’écran illustrant votre estimation](media/storage-plan-manage-costs/price-calculator-storage-type.png)

   Quand vous modifiez la valeur de la liste déroulante **Type**, d’autres options de cette feuille de calcul changent également. Utilisez les liens de la section **Plus d’informations** pour en savoir plus sur la fonction de chaque option et sur la façon dont ces options affectent le prix des opérations liées au stockage. 

4. Modifiez les options restantes pour voir leur impact sur votre estimation.

   > [!NOTE]
   > Vous pouvez payer les frais de Stockage Blob Azure avec votre crédit d’engagement financier du Contrat Entreprise Azure. Vous ne pouvez cependant pas utiliser le crédit d’engagement financier pour payer des frais pour des produits et services tiers, y compris ceux de la Place de marché Azure.

## <a name="optimize-costs"></a>Optimiser les coûts

Envisagez d’utiliser ces options pour réduire les coûts. 

- Réserver de la capacité de stockage

- Organiser les données en niveaux d’accès

- Déplacer automatiquement les données entre les niveaux d’accès

Cette section décrit chaque option plus en détail. 

#### <a name="reserve-storage-capacity"></a>Réserver de la capacité de stockage

Vous pouvez réaliser des économies sur les coûts de stockage pour les données blob grâce à une capacité de réserve Stockage Azure. La capacité de réserve Stockage Azure offre une remise sur la capacité pour les objets blob de blocs et les données Azure Data Lake Storage Gen2 dans les comptes de stockage standard lorsque vous vous engagez à une réservation pour un an ou trois ans. Une réservation offre une quantité fixe de capacité de stockage pour la durée de la réservation. La capacité de réserve Stockage Azure peut réduire de manière significative les coûts de capacité pour les objets blob de blocs et les données Azure Data Lake Storage Gen2. 

Pour en savoir plus, consultez [Optimiser les coûts de stockage d’objets blob avec une capacité réservée](../blobs/storage-blob-reserved-capacity.md).

#### <a name="organize-data-into-access-tiers"></a>Organiser les données en niveaux d’accès

Vous pouvez réduire les coûts en plaçant les données blob dans les niveaux d’accès les plus rentables. Choisissez parmi trois niveaux conçus pour optimiser vos coûts d’utilisation des données. Par exemple, le niveau d’accès *chaud* présente un coût de stockage plus élevé, mais un coût d’accès inférieur. Par conséquent, si vous envisagez d’accéder fréquemment aux données, le niveau d’accès chaud peut être le choix le plus rentable. Si vous prévoyez d’accéder moins fréquemment aux données, le niveau d’accès *froid* ou *archive* pourrait être le mieux adapté, car il augmente le coût d’accès aux données tout en réduisant le coût de stockage des données.    

Pour en savoir plus, consultez [Stockage Blob Azure : niveaux d’accès chaud, froid et archive](../blobs/storage-blob-storage-tiers.md?tabs=azure-portal).

#### <a name="automatically-move-data-between-access-tiers"></a>Déplacer automatiquement les données entre les niveaux d’accès

Utilisez les stratégies de gestion du cycle de vie pour déplacer périodiquement les données entre les niveaux afin d’effectuer le plus d’économies possible. Ces stratégies peuvent déplacer des données vers à l’aide des règles que vous indiquez. Par exemple, vous pouvez créer une règle qui déplace les objets blob vers le niveau Archive si cet objet blob n’a pas été modifié dans les 90 jours. En créant des stratégies qui ajustent le niveau d’accès de vos données, vous pouvez concevoir les options de stockage les moins onéreuses pour vos besoins.

Pour plus d’informations, consultez [Gérer le cycle de vie du service Stockage Blob Azure](../blobs/storage-lifecycle-management-concepts.md?tabs=azure-portal)

## <a name="create-budgets"></a>Créer des budgets

Vous pouvez créer des [budgets](../../cost-management-billing/costs/tutorial-acm-create-budgets.md) pour gérer les coûts, et des alertes permettant d’avertir automatiquement les parties prenantes en cas d’anomalies de dépenses et de risques de dépenses excessives. Les alertes sont basées sur les dépenses par rapport aux seuils de budget et de coût. Les budgets et les alertes sont créés pour les abonnements et les groupes de ressources Azure : ils sont donc utiles dans le cadre d’une stratégie globale de supervision des coûts. Ils peuvent cependant avoir des fonctionnalités limitées quant à la gestion des coûts de services Azure individuels comme le coût du stockage Azure. En effet, ils sont conçus pour suivre les coûts à un niveau supérieur.

## <a name="monitor-costs"></a>Superviser les coûts

À mesure que vous utilisez des ressources Azure avec le stockage Azure, vous générez des coûts. Les coûts unitaires d’utilisation des ressources varient selon les intervalles de temps (secondes, minutes, heures et jours) ou selon l’utilisation d’unités (octets, mégaoctets, etc.). Des coûts sont générés dès que vous commencez à utiliser le stockage Azure. Vous pouvez voir les coûts dans le volet [Analyse du coût](../../cost-management-billing/costs/quick-acm-cost-analysis.md) du portail Azure.

Quand vous utilisez l’analyse du coût, vous pouvez voir les coûts du stockage Azure dans des graphes et des tableaux pour différents intervalles de temps. Par exemple, par jour, par année ou pour le mois en cours ou le mois précédent. Vous pouvez également voir les coûts par rapport aux budgets et aux coûts prévus. Passez à des vues pour des périodes plus longues pour identifier les tendances des dépenses et déterminer où des dépassements ont pu se produire. Par ailleurs, si vous avez créé des budgets, vous pouvez facilement voir à quel moment ils ont été dépassés.

>[!NOTE]
> L’analyse des coûts prend en charge différents types de comptes Azure. Pour accéder à la liste complète des types de comptes pris en charge, voir [Comprendre les données de Cost Management](../../cost-management-billing/costs/understand-cost-mgt-data.md). Pour afficher les données de coût, vous avez au minimum besoin d’un accès en lecture pour votre compte Azure. Pour plus d’informations sur l’attribution de l’accès aux données Azure Cost Management, consultez [Assigner l’accès aux données](../../cost-management-billing/costs/assign-access-acm-data.md).

Pour voir les coûts du stockage Azure dans l’analyse du coût :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Ouvrez la fenêtre **Gestion des coûts + Facturation**, sélectionnez **Gestion des coûts** dans le menu, puis sélectionnez **Analyse des coûts**. Vous pouvez ensuite changer l’étendue pour un abonnement spécifique dans la liste déroulante **Étendue**.

   ![Capture d’écran illustrant la portée](./media/storage-plan-manage-costs/cost-analysis-pane.png)

4. Pour voir uniquement les coûts du stockage Azure, sélectionnez **Ajouter un filtre**, puis sélectionnez **Nom du service**. Ensuite, choisissez **stockage** dans la liste. 

   Voici un exemple montrant les coûts du stockage Azure uniquement :

   ![Capture d’écran illustrant le filtre par stockage](./media/storage-plan-manage-costs/cost-analysis-pane-storage.png)

Dans l’exemple précédent, vous avez pu voir le coût actuel du service. Les coûts par région Azure (emplacements) et par groupe de ressources sont également présentés. Vous pouvez également ajouter d’autres filtres (par exemple : un filtre pour afficher les coûts de comptes de stockage spécifiques).

## <a name="export-cost-data"></a>Exporter des données de coûts

Vous pouvez également [exporter vos données de coûts](../../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) vers un compte de stockage. C’est utile quand vous ou d’autres personnes avez besoin d’effectuer des analyses supplémentaires des données concernant les coûts. Par exemple, une équipe Finance peut analyser les données avec Excel ou Power BI. Vous pouvez exporter vos coûts selon une planification quotidienne, hebdomadaire ou mensuelle, et définir une plage de dates personnalisée. L’exportation des données des coûts est la méthode recommandée pour récupérer les jeux de données des coûts.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus en détail comment fonctionnent les tarifs avec Stockage Azure. Consultez [Vue d’ensemble de la tarification de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/).
- [Optimisez les coûts pour Stockage Blob avec une capacité réservée](../blobs/storage-blob-reserved-capacity.md).
- Découvrez [comment optimiser votre investissement cloud avec Azure Cost Management](../../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Apprenez-en davantage sur la gestion des coûts avec l’[analyse du coût](../../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Découvrez comment [éviter des coûts imprévus](../../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Suivez le cours d’apprentissage guidé [Cost Management](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).