---
title: Déterminer la réservation Azure à acheter
description: Cet article vous permet de déterminer la réservation à acheter.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 08/04/2020
ms.author: banders
ms.openlocfilehash: d89f890d3e2bb8238a00b4a529b8804a8c38fa6f
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684743"
---
# <a name="determine-what-reservation-to-purchase"></a>Déterminer la réservation à acheter

Toutes les réservations s’appliquent sur une base horaire sauf pour Azure Databricks. Vous devez acheter des réservations sur la base d’une utilisation systématique. Vous pouvez déterminer ce que vous devez acheter de plusieurs façons, et cet article vous permet de déterminer la réservation à acheter.

Si vous achetez une capacité supérieure à votre historique d’utilisation, votre réservation sera sous-utilisée. Vous devez éviter une sous-utilisation autant que possible. Une capacité de réserve inutilisée n’est pas reportée sur l’heure suivante. Une utilisation dépassant la quantité réservée est facturée au tarif du paiement à l’utilisation, qui est plus élevé.

## <a name="analyze-usage-data"></a>Analyser les données d’utilisation

Les sections suivantes vous permettent d’analyser votre utilisation de données quotidienne et ainsi de déterminer votre utilisation de référence et la réservation à acheter.

### <a name="analyze-usage-for-a-vm-reserved-instance-purchase"></a>Analyser l’utilisation en vue de l’achat d’instances de machine virtuelle réservées

Identifiez la taille de machine virtuelle appropriée pour votre achat. Par exemple, une réservation achetée pour des machines virtuelles de la série ES ne s’applique pas aux machines virtuelles de la série E et inversement.

Les machines virtuelles de la série Promo ne sont pas éligibles aux remises sur réservation. Vous devez donc les retirer de votre analyse.

Pour limiter l’analyse à l’utilisation de machines virtuelles éligibles, appliquez les filtres suivants à vos données d’utilisation :

- Filtrez **Catégorie du compteur** sur **Machines virtuelles**.
- Récupérez les informations de **Type de service** dans **Informations supplémentaires**. Ces informations donnent des indications sur la taille de machine virtuelle appropriée. Par exemple, Standard E32.
- Utilisez le champ **Emplacement des ressources** pour déterminer le centre de données d’utilisation.

Ignorez les ressources dont l’utilisation est inférieure à 24 heures par jour.

Si vous souhaitez effectuer l’analyse au niveau de la famille de tailles d’instance, vous pouvez obtenir les valeurs de flexibilité de taille d’instance à partir de [https://isfratio.blob.core.windows.net/isfratio/ISFRatio.csv](https://isfratio.blob.core.windows.net/isfratio/ISFRatio.csv). Combinez les valeurs avec vos données pour effectuer l’analyse. Pour plus d’informations à propos de la flexibilité de la taille d’instance, consultez [Flexibilité en termes de taille de machine virtuelle avec des instances de machines virtuelles réservées](../../virtual-machines/reserved-vm-instance-size-flexibility.md).

### <a name="analyze-usage-for-an-azure-synapse-analytics-reserved-instance-purchase"></a>Analyser l’utilisation en vue d’acheter des instances Azure Synapse Analytics réservées

La capacité de réserve s’applique aux tarifs DWU d’Azure Synapse Analytics. Elle ne s’applique pas au coût des licences Azure Synapse Analytics et aux coûts autres que le calcul.

Pour limiter l’utilisation éligible, appliquez les filtres suivants à vos données d’utilisation :


- Filtrez **Catégorie du compteur** sur **Base de données SQL**.
- Filtrez **Nom du compteur** sur **vCore**.
- Filtrez **Sous-catégorie du compteur** sur tous les enregistrements d’utilisation dont le nom contient _Compute_.

Récupérez la valeur **vCores** dans **Informations supplémentaires**. Elle indique le nombre de vCores utilisés. La quantité correspond à **vCores** multiplié par le nombre d’heures d’utilisation de la base de données.

Les données vous informent sur l’utilisation systématique selon différents aspects :

- Combinaison du type de base de données. Par exemple, instance managée ou pool élastique par base de données.
- Niveau de service. Par exemple, usage général ou vital pour l’entreprise.
- Génération. Par exemple, Gen 5.
- Emplacement de la ressource

### <a name="analysis-for-azure-synapse-analytics"></a>Analyse pour Azure Synapse Analytics

La capacité de réserve s’applique à l’utilisation de Data Warehouse Units (DWU) pour Azure Synapse Analytics et s’achète par incréments de 100 DWU. Pour limiter l’utilisation éligible, appliquez les filtres suivants à vos données d’utilisation :

- Filtrez **Nom du compteur** sur **100 DWU**.
- Filtrez **Sous-catégorie du compteur** sur **Calcul optimisé Gen2**.

Utilisez le champ **Emplacement de la ressource** pour déterminer l’utilisation d’Azure Synapse Analytics dans une région.

L’utilisation d’Azure Synapse Analytics peut évoluer à la hausse et à la baisse tout au long de la journée. Communiquez avec l’équipe qui a géré l’instance Azure Synapse Analytics pour obtenir des informations sur l’utilisation de base.

Accédez à Réservations dans le portail Azure et achetez la capacité de réserve d’Azure Synapse Analytics par multiples de 100 DWU.

## <a name="reservation-purchase-recommendations"></a>Recommandations relatives à l’achat de réservation

Les recommandations relatives à l’achat de réservation sont calculées sur la base de l’analyse de vos données d’utilisation horaires sur les 7, 30 et 60 derniers jours. Azure calcule les coûts que vous auriez encourus si vous aviez eu une réservation et les compare aux coûts réels du paiement à l’utilisation sur la période. Le calcul est effectué pour chaque quantité que vous avez utilisée au cours de la période. La quantité qui offrirait des économies optimales est recommandée.

Par exemple, il se peut que vous utilisiez 500 machines virtuelles la plupart du temps, mais que votre utilisation présente parfois des pics s’élevant à 700 machines virtuelles. Dans ce cas, Azure calcule vos économies pour les deux quantités de machines virtuelles : 500 et 700. Étant donné que l’utilisation de 700 machines virtuelles est sporadique, le calcul de la recommandation détermine que les économies sont maximisées pour un achat de réservation de 500 machines virtuelles. La recommandation indique donc une quantité de 500 machines virtuelles.

Notez les points suivants :

- Les recommandations de réservation sont calculées à partir des taux d’utilisation à la demande qui s’appliquent à votre situation.
- Les recommandations sont calculées pour des tailles individuelles et non pour la famille de tailles d’instance.
- La quantité recommandée pour une étendue est réduite le jour où vous achetez des réservations pour l’étendue.
    - Toutefois, une mise à jour de la recommandation de quantité de réservation selon les étendues peut prendre jusqu’à 25 jours. Par exemple, si vous effectuez votre achat en fonction de recommandations portant sur une étendue partagée, les recommandations portant sur une étendue d’abonnement unique peuvent prendre jusqu’à 25 jours pour indiquer la quantité réduite.

## <a name="recommendations-in-the-azure-portal"></a>Recommandations dans le portail Azure

Les achats de réservation calculés par le moteur de recommandations apparaissent sous l’onglet **Recommandé** du [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/docs). Cette image en fournit un exemple.

![Image présentant les recommandations](./media/determine-reservation-purchase/select-product-ri.png)

Apprenez-en davantage sur les [recommandations](reserved-instance-purchase-recommendations.md#recommendations-in-the-azure-portal).

## <a name="recommendations-in-the-cost-management-power-bi-app"></a>Recommandations dans l’application Cost Management pour Power BI

Les clients Accord Entreprise et Contrat client Microsoft peuvent utiliser les rapports de couverture des instances de machine virtuelle réservées et les recommandations d’achat. Les rapports de couverture indiquent l’utilisation totale et l’utilisation couverte par les instances réservées.

1. Procurez-vous l’[application Cost Management](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp).
2. Accédez au rapport de couverture des instances de machine virtuelle réservées. Choisissez une étendue unique ou partagée selon l’étendue faisant l’objet de l’achat.
3. Sélectionnez la région, la famille de tailles d’instance pour voir l’utilisation, la couverture des instances réservées et la recommandation d’achat pour le filtre sélectionné.

## <a name="recommendations-in-azure-advisor"></a>Recommandations dans Azure Advisor

Les recommandations d’achat de réservation sont disponibles dans [Azure Advisor](https://portal.azure.com/#blade/Microsoft_Azure_Expert/AdvisorMenuBlade/overview).

- Advisor produit des recommandations dont l’étendue est limitée à un seul abonnement.
- Les recommandations d’Advisor sont calculées sur la base d’une période antérieure de 30 jours. Les économies prévues portent sur une durée de réservation de 3 ans.
- Si vous achetez une réservation d’étendue partagée, les recommandations d’achat de réservations d’Advisor peuvent prendre jusqu’à 30 jours pour disparaître.

## <a name="recommendations-using-apis"></a>Recommandations à l’aide d’API

Utilisez l’API REST [Reservation Recommendations](/rest/api/consumption/reservationrecommendations/list) pour voir les recommandations programmatiquement.

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les réservations Azure](manage-reserved-vm-instance.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
- [Coûts des logiciels Windows non inclus dans les réservations](reserved-instance-windows-software-costs.md)