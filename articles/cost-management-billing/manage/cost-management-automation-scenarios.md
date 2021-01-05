---
title: Scénarios d’automatisation de la facturation et de la gestion des coûts Azure
description: Découvrez comment des scénarios de facturation et de gestion des coûts courants sont mappés vers différentes API.
author: bandersmsft
ms.reviewer: adwise
tags: billing
ms.service: cost-management-billing
ms.subservice: common
ms.topic: reference
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: 8cef36cf26231758eb839379d0c020484cbe09fb
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132599"
---
# <a name="automation-scenarios-for-billing-and-cost-management"></a>Scénarios d’automatisation de la facturation et de la gestion des coûts

Cet article liste les scénarios courants de facturation et de gestion des coûts Azure. Ces scénarios sont mis en correspondance avec les API que vous pouvez utiliser. Sous chaque correspondance entre un scénario et une API, vous trouverez un résumé des API et des fonctionnalités qu’elles proposent.

## <a name="common-scenarios"></a>Scénarios courants

Vous pouvez utiliser les API de facturation et de gestion des coûts dans plusieurs scénarios pour répondre aux questions liées aux coûts et à l’utilisation. Voici une brève présentation des scénarios courants :

- **Rapprochement de facture** : Est-ce que Microsoft m’a facturé le bon montant ?  À combien ma facture s’élève-t-elle et puis-je la calculer moi-même ?

- **Répartition interne** : Maintenant que je sais à combien s’élève ma facture, qui doit payer au sein de mon organisation ?

- **Optimisation des coûts** : Je sais à combien s’élève ma facture. Comment puis-je rentabiliser davantage l’argent que je dépense dans Azure ?

- **Suivi des coûts** : Je veux voir combien je dépense et comment j’utilise Azure sur une période donnée. Quelles sont les tendances ? Comment puis-je faire mieux ?

- **Dépenses Azure au cours du mois** : Combien ai-je dépensé ce mois-ci jusqu’à aujourd’hui ? Dois-je procéder à quelques modifications dans mes dépenses et/ou mon utilisation d’Azure ? Quand, dans le mois, ma consommation d’Azure est-elle la plus élevée ?

- **Alertes** : Comment puis-je configurer la consommation en fonction des ressources ou des alertes monétaires ?

## <a name="scenario-to-api-mapping"></a>Correspondance entre scénario et API

|         API        | Rapprochement de facture    | Répartition interne    | Optimisation des coûts    | Suivi des coûts    | Dépenses en cours de mois    | Alertes    |
|:---------------------------:|:-------------------------:|:----------------:|:--------------------:|:----------------:|:------------------:|:---------:|
| Budgets                     |                           |                  |           X          |                  |                    |     X     |
| Marketplace Charges                |             X             |         X        |           X          |         X        |          X         |     X     |
| Price Sheet                 |             X             |         X        |           X          |         X        |          X         |           |
| Reservation Recommendations |                           |                  |           X          |                  |                    |           |
| Reservation Details         |                           |                  |           X          |         X        |                    |           |
| Reservation Summaries       |                           |                  |           X          |         X        |                    |           |
| Usage Details               |             X             |         X        |           X          |         X        |          X         |     X     |
| Billing Periods             |             X             |         X        |           X          |         X        |                    |           |
| Factures                    |             X             |         X        |           X          |         X        |                    |           |
| RateCard                    |             X             |                  |           X          |         X        |          X         |           |
| Unrated Usage               |             X             |                  |           X          |                  |          X         |           |

> [!NOTE]
> Les correspondances entre scénario et API n’incluent pas les API Enterprise Consumption. Si possible, utilisez les API Consumption générales pour les nouveaux scénarios de développement.

## <a name="api-summaries"></a>Résumés des API

### <a name="consumption"></a>Consommation
Les clients Web Direct et Entreprise peuvent utiliser toutes les API suivantes, sauf indication contraire :

-   [API Budgets](/rest/api/consumption/budgets) (*clients Entreprise uniquement*) : Créez des budgets pour les coûts ou l’utilisation des ressources, groupes de ressources ou compteurs de facturation. Avec des budgets, vous pouvez configurer des alertes pour être averti lorsque vous en avez dépassé les limites définies. Vous pouvez également configurer des actions à effectuer quand vous avez atteint les montants des budgets.

-   [API Marketplace Charges](/rest/api/consumption/marketplaces) : Obtenez des données de paiement et d’utilisation sur toutes les ressources de la Place de marché Azure (offres des partenaires Azure). Vous pouvez utiliser ces données pour additionner les coûts de toutes les ressources de la Place de marché ou pour examiner les coûts/l’utilisation de ressources spécifiques.

-   [API Price Sheet](/rest/api/consumption/pricesheet) (*Clients Entreprise uniquement*) : Obtenez des prix personnalisés pour tous les compteurs. Les entreprises peuvent utiliser ces données conjointement avec les informations sur l’utilisation pour calculer des coûts.

-   [API Reservation Recommendations](/rest/api/consumption/reservationrecommendations) : Obtenez des recommandations d’achat d’instances de machine virtuelle réservées. Ces recommandations vous permettent d’analyser les économies et les montants d’achat attendus. Pour plus d’informations, consultez [API dédiées à l’automatisation de la réservation Azure](../reservations/reservation-apis.md).

-   [API Reservation Details](/rest/api/consumption/reservationsdetails) : Consultez les informations sur les réservations de machines virtuelles précédemment achetées, notamment sur la consommation réservée par rapport à la quantité utilisée. Vous pouvez voir les données au niveau de la machine virtuelle. Pour plus d’informations, consultez [API dédiées à l’automatisation de la réservation Azure](../reservations/reservation-apis.md).

-   [API Reservation Summaries](/rest/api/consumption/reservationssummaries) : Consultez les informations agrégées sur les réservations de machines virtuelles que votre organisation a achetées, comme la consommation réservée par rapport à la quantité utilisée dans l’agrégat. Pour plus d’informations, consultez [API dédiées à l’automatisation de la réservation Azure](../reservations/reservation-apis.md).

-   [API Usage Details](/rest/api/consumption/usagedetails) : Obtenez des informations de paiement et d’utilisation sur toutes les ressources Azure de Microsoft. Les informations sont sous la forme d’enregistrements détaillés de l’utilisation, qui sont actuellement émis une fois par compteur par jour. Vous pouvez utiliser ces informations pour additionner les coûts de toutes les ressources ou examiner les coûts/l’utilisation de ressources spécifiques.

-   [API RateCard](/previous-versions/azure/reference/mt219005(v=azure.100)) : Obtenez des tarifs spéciaux si vous êtes client Web Direct. Vous pouvez alors utiliser les informations retournées avec vos informations sur l’utilisation de la ressource pour calculer manuellement la facture prévue.

-   [API Unrated Usage](/previous-versions/azure/reference/mt219003(v=azure.100)) : Obtenez des informations sur l’utilisation brute avant tout décompte/toute facturation Azure.

### <a name="billing"></a>Facturation
-   [API Billing Periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) : Déterminez une période de facturation à analyser, ainsi que les ID de facture de cette période. Vous pouvez utiliser des ID de facture avec l’API Invoices.

-   [API Invoices](/rest/api/billing/2019-10-01-preview/invoices) : Obtenez l’URL de téléchargement d’une facture pour une période de facturation au format PDF.

### <a name="enterprise-consumption"></a>Consommation de l’entreprise
Les API suivantes s’appliquent uniquement aux clients Entreprise :

-   [API Balance Summary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) : Obtenez une synthèse mensuelle des informations sur les soldes, nouveaux achats, frais de service de la Place de marché Azure, ajustements et dépassement de frais. Vous pouvez obtenir ces informations pour la période de facturation en cours ou n’importe quelle période passée. Les entreprises peuvent utiliser ces données pour les comparer avec les frais résumés calculés. Cette API ne fournit pas d’informations propres à la ressource ou une vue agrégée des coûts.

-   [API Usage Details](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail) : Obtenez des informations sur l’utilisation Azure (des offres Microsoft) pendant le mois en cours, une période de facturation spécifique ou une plage de dates personnalisée. Les entreprises peuvent utiliser ces données pour calculer manuellement des factures en fonction de leurs tarifs et consommation. Elles peuvent également utiliser les informations sur les départements/organisations pour imputer les coûts à toutes les organisations. Les données fournissent une vue propre à la ressource de l’utilisation/des coûts.

-   [API Marketplace Store Charge](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge) : Obtenez des informations sur l’utilisation Azure (des offres de partenaires) pendant le mois en cours, une période de facturation spécifique ou une plage de dates personnalisée. Les entreprises peuvent utiliser ces données pour calculer manuellement des factures en fonction de leurs tarifs et consommation. Elles peuvent également utiliser les informations sur les départements/organisations pour imputer les coûts à toutes les organisations. Cette API fournit un aperçu propre à la ressource de l’utilisation/des coûts.

-   [API Price Sheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) : Obtenez les tarifs applicables à chaque compteur pendant la période d’inscription et de facturation donnée. Vous pouvez utiliser ces informations tarifaires en combinaison avec celles sur l’utilisation pour calculer manuellement la facture prévue.

-   [API Billing Periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) : Obtenez la liste des périodes de facturation. Cette API vous donne également une propriété qui pointe vers la route d’API des quatre jeux de données d’API Enterprise qui se rapportent à la période de facturation : BalanceSummary, UsageDetails, Marketplace Charges et PriceSheet.

-   [API Reserved Instance Recommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation) : Examinez l’utilisation des machines virtuelles sur 7 jours, 30 jours ou 60 jours et obtenez des recommandations d’achat de machines uniques et partagées. Vous pouvez utiliser cette API pour analyser les économies et les montants d’achat attendus. Pour plus d’informations, consultez [API dédiées à l’automatisation de la réservation Azure](../reservations/reservation-apis.md).

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="whats-the-difference-between-the-enterprise-reporting-apis-and-the-consumption-apis-when-should-i-use-each"></a>Quelle est la différence entre les API Enterprise Reporting et Consumption ? Quand utiliser chaque API ?
Ces API offrent un ensemble similaire de fonctionnalités et peuvent répondre à la même série de questions dans les domaines de la facturation et de la gestion des coûts. Mais elles ciblent différents publics :

- Les API Enterprise Reporting sont accessibles aux clients qui ont signé un Contrat Entreprise avec Microsoft. Ce contrat leur donne accès à des engagements monétaires négociés et à des tarifs personnalisés. Les API nécessitent une clé que vous pouvez obtenir depuis [Enterprise Portal](https://ea.azure.com). Pour obtenir une description de ces API, consultez [Vue d’ensemble des API Reporting pour les clients Enterprise](enterprise-api.md).

- Les API Consumption sont accessibles à tous les clients, à quelques exceptions près. Pour plus d’informations, consultez [Vue d’ensemble des API Azure Consumption](consumption-api-overview.md) et [Informations de référence sur les API Azure Consumption](/rest/api/consumption/). Nous recommandons les API fournies comme solution pour les scénarios de développement les plus récents.

### <a name="whats-the-difference-between-the-usage-details-api-and-the-usage-api"></a>Quelle est la différence entre les API Usage Details et Usage ?
Ces API fournissent des données fondamentalement différentes :

- [l’API Usage Details](/rest/api/consumption/usagedetails) fournit des informations relatives au coût et à l’utilisation d’Azure par instance de compteur. Les données fournies sont déjà passées par le système de comptage des coûts dans Azure, ont des coûts associés et reflètent d’autres changements éventuels :

   - Changements pour tenir compte de l’utilisation des engagements monétaires prépayés
   - Changements pour tenir compte des différences d’utilisation découvertes par Azure

- L’[API Usage](/previous-versions/azure/reference/mt219003(v=azure.100)) fournit des informations brutes sur l’utilisation d’Azure avant qu’elles ne passent par le système de comptage des coûts dans Azure. Ces données peuvent ne présenter aucune corrélation avec l’utilisation ou les frais constatés après leur passage dans le système de comptage des coûts d’Azure.

### <a name="whats-the-difference-between-the-invoice-api-and-the-usage-details-api"></a>Quelle est la différence entre les API Invoice et Usage Details ?
Ces API offrent un aperçu différent des mêmes données :

- L’[API Invoice](/rest/api/billing/2019-10-01-preview/invoices) concerne uniquement les clients Web Direct. Elle fournit un cumul mensuel de votre facture en fonction des frais agrégés de chaque type de compteur.

- L’[API Usage Details](/rest/api/consumption/usagedetails) fournit un aperçu plus précis des enregistrements quotidiens des coûts/de l’utilisation. Les clients Web Direct et Entre²prise peuvent l’utiliser.

### <a name="whats-the-difference-between-the-price-sheet-api-and-the-ratecard-api"></a>Quelle est la différence entre les API Price Sheet et RateCard ?
Ces API fournissent des jeux de données similaires, mais concernent des publics différents :

- [l’API Price Sheet](/rest/api/consumption/pricesheet) fournit les tarifs personnalisés qui ont été négociés pour un client Enterprise.

- L’[API RateCard](/previous-versions/azure/reference/mt219005(v=azure.100)) fournit les tarifs publics applicables aux clients Web Direct.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’utilisation des API Azure pour obtenir par programmation des insights sur votre utilisation d’Azure, consultez [Présentation des API Azure Consumption](consumption-api-overview.md) et [Présentation des API Azure Billing](usage-rate-card-overview.md).

- Pour comparer votre facture au fichier détaillé de l’utilisation quotidienne et aux rapports de gestion des coûts du portail Azure, consultez [Comprendre votre facture pour Microsoft Azure](../understand/review-individual-bill.md).

- Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).