---
title: Comprendre l’utilisation des réservations Azure pour les Contrats Entreprise
description: Découvrez comment analyser votre utilisation pour comprendre l’application de la réservation Azure pour votre Accord de Mise en Œuvre Entreprise.
author: bandersmsft
ms.reviewer: yashar
tags: billing
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: banders
ms.openlocfilehash: 874d5cb022a38b172bb37009bd86b5e6988f3204
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545602"
---
# <a name="get-enterprise-agreement-reservation-costs-and-usage"></a>Obtenir les données d’utilisation et de coûts de la réservation pour les Contrats Entreprise

Les données d’utilisation et de coûts de la réservation sont disponibles pour les clients Contrat Entreprise dans le portail Azure et les API REST. Cet article vous aide à :

- Obtenir les données d’achat de réservation
- Connaître l’abonnement, le groupe de ressources ou la ressource qui a utilisé la réservation
- Rétrofacturer l’utilisation des réservations
- Calculer les économies des réservations
- Obtenir les données de sous-utilisation des réservations
- Amortir les coûts des réservations

Les frais de la marketplace sont consolidés dans les données d’utilisation. Vous voyez les frais liés à l’utilisation interne, à l’utilisation de la marketplace et aux achats à partir d’une source de données unique.

## <a name="reservation-charges-in-azure-usage-data"></a>Frais de réservation dans les données d’utilisation Azure

Les données sont divisées en deux jeux de données distincts : _Coût réel_ et _Coût amorti_. Comparons ces deux jeux de données :

**Coût réel** : fournit les données à rapprocher avec votre facture mensuelle. Ces données ont des coûts d’achat de réservation et des détails sur l’application de la réservation. Avec ces données, vous pouvez savoir quel abonnement ou groupe de ressources ou ressource a reçu la remise de réservation un jour particulier. Le prix réel (EffectivePrice) pour l’utilisation qui a bénéficié de la remise de réservation est égal à zéro.

**Coût amorti** : ce jeu de données est similaire au jeu de données Coût réel, sauf que le prix réel (EffectivePrice) pour l’utilisation qui obtient une remise de réservation correspond au coût au prorata de la réservation (au lieu de zéro). Cela vous permet de connaître la valeur monétaire de la consommation de réservation d’un abonnement, d’un groupe de ressources ou d’une ressource, et peut vous aider à refacturer en interne pour l’utilisation de la réservation. Le jeu de données contient également les heures inutilisées de la réservation. Le jeu de données n’a pas d’enregistrements d’achat de réservation.

Comparaison des deux jeux de données :

| Données | Jeu de données du coût réel | Jeu de données du coût amorti |
| --- | --- | --- |
| Achats de réservation | Disponible dans cet affichage.<br><br>  Pour obtenir ces données, appliquez le filtre ChargeType = &quot;Purchase&quot;. <br><br> Reportez-vous à ReservationID ou ReservationName pour savoir à quelle réservation correspondent les frais.  | Non applicables à cet affichage. <br><br> Les coûts d’achat ne sont pas fournis dans les données amorties. |
| EffectivePrice | La valeur est égale à zéro pour l’utilisation qui bénéficie de la remise de réservation. | La valeur est le coût horaire au prorata de la réservation pour l’utilisation qui bénéficie de la remise de réservation. |
| Réservation inutilisée (fournit le nombre d’heures pendant lesquelles la réservation n’a pas été utilisée dans une journée et la valeur monétaire de la perte) | Non applicable dans cet affichage. | Disponible dans cet affichage.<br><br> Pour obtenir ces données, appliquez le filtre ChargeType = &quot;UnusedReservation&quot;.<br><br>  Reportez-vous à ReservationID ou ReservationName pour savoir quelle réservation a été sous-utilisée. Il s’agit de la quantité de la réservation qui a été perdue au cours de la journée.  |
| UnitPrice (prix de la ressource d’après votre grille tarifaire) | Disponible | Disponible |

Les autres informations disponibles dans les données d’utilisation Azure ont changé :

- Informations sur les produits et les compteurs : Azure ne remplace plus le compteur initialement consommé par ReservationId et ReservationName.
- ReservationId et ReservationName : ils ont leurs propres champs dans les données. Auparavant, ils étaient disponibles uniquement sous AdditionalInfo.
- ProductOrderId : ID d’ordre de réservation, ajouté en tant que champ spécifique.
- ProductOrderName : nom de produit de la réservation achetée.
- Term : 12 mois ou 36 mois.
- RINormalizationRatio : disponible sous AdditionalInfo. Il s’agit du taux d’application de la réservation à l’enregistrement d’utilisation. Si la flexibilité de la taille d’instance est activée pour votre réservation, elle peut s’appliquer à d’autres tailles. La valeur indique le taux d’application de la réservation à l’enregistrement d’utilisation.

[Consulter la définition des champs](/rest/api/consumption/usagedetails/list#definitions)

## <a name="get-azure-consumption-and-reservation-usage-data-using-api"></a>Obtenir les données d’utilisation des réservations et de consommation Azure à l’aide de l’API

Vous pouvez obtenir les données à l’aide de l’API ou les télécharger à partir du portail Azure.

Vous appelez l’[API Détails d’utilisation](/rest/api/consumption/usagedetails/list) pour obtenir les nouvelles données. Pour plus d’informations sur la terminologie, consultez [Termes relatifs à l’utilisation](../understand/understand-usage.md). L’appelant doit être administrateur d’entreprise pour le contrat entreprise utilisant le [portail EA](https://ea.azure.com). Les administrateurs d’entreprise disposant d’un accès en lecture seule peuvent également obtenir les données.

Notez que ces données ne sont pas disponibles dans [API de création de rapports pour les clients Enterprise - Détails d’utilisation](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail).

Voici un exemple d’appel de l’API Détails d’utilisation :

```
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{enrollmentId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodId}/providers/Microsoft.Consumption/usagedetails?metric={metric}&amp;api-version=2019-05-01&amp;$filter={filter}
```

Pour plus d’informations sur {enrollmentId} et {billingPeriodId}, consultez l’article sur l’API [Détails d’utilisation - Liste](/rest/api/consumption/usagedetails/list).

Les informations contenues dans le tableau suivant sur metric (métrique) et filter (filtre) peuvent aider à résoudre les problèmes courants liés aux réservations.

| **Type de données de l’API** | Action d’appel d’API |
| --- | --- |
| **Tous les frais (utilisation et achats)** | Remplacez {metric} par ActualCost |
| **Utilisation qui a bénéficié d’une remise de réservation** | Remplacez {metric} par ActualCost<br><br>Remplacez {filter} par properties/reservationId%20ne%20 |
| **Utilisation qui n’a pas bénéficié d’une remise de réservation** | Remplacez {metric} par ActualCost<br><br>Remplacez {filter} par properties/reservationId%20eq%20 |
| **Frais amortis (utilisation et achats)** | Remplacez {metric} par AmortizedCost |
| **Rapport sur la réservation inutilisée** | Remplacez {metric} par AmortizedCost<br><br>Remplacez {filter} par properties/ChargeType%20eq%20'UnusedReservation' |
| **Achats de réservation** | Remplacez {metric} par ActualCost<br><br>Remplacez {filter} par properties/ChargeType%20eq%20'Purchase'  |
| **Remboursements** | Remplacez {metric} par ActualCost<br><br>Remplacez {filter} par properties/ChargeType%20eq%20'Refund' |

## <a name="download-the-usage-csv-file-with-new-data"></a>Télécharger le fichier CSV d’utilisation avec les nouvelles données

Si vous êtes administrateur EA, vous pouvez télécharger le fichier CSV qui contient les nouvelles données d’utilisation à partir du portail Azure. Ces données n’étant pas disponibles à partir du portail EA (ea.azure.com), vous devez télécharger le fichier d’utilisation à partir du portail Azure (portal.azure.com) pour afficher les nouvelles données.

Dans le portail Azure, accédez à [Gestion des coûts + facturation](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/BillingAccounts).

1. Sélectionnez le compte de facturation.
2. Cliquer sur **Utilisation + frais**.
3. Cliquez sur **Télécharger**.  
![Exemple montrant où télécharger le fichier CSV des données d’utilisation dans le portail Azure](./media/understand-reserved-instance-usage-ea/portal-download-csv.png)
4. Dans **Télécharger l’utilisation + les frais**, sous **Détails d’utilisation version 2**, sélectionnez **Tous les frais (utilisation et achats)** puis cliquez sur Télécharger. Répétez la procédure pour **Frais amortis (utilisation et achats)** .

Les fichiers CSV que vous téléchargez contiennent les coûts réels et les coûts amortis.

## <a name="common-cost-and-usage-tasks"></a>Tâches courantes liées aux coûts et à l’utilisation

Les sections suivantes décrivent les tâches couramment utilisées pour afficher les données d’utilisation et de coût des réservations.

### <a name="get-reservation-purchase-costs"></a>Obtenir les coûts d’achat de réservation

Les coûts d’achat de réservation sont disponibles dans les données Coût réel. Appliquez le filtre _ChargeType = Purchase_. Reportez-vous à ProductOrderID pour déterminer quel ordre de réservation est concerné par l’achat.

### <a name="get-underutilized-reservation-quantity-and-costs"></a>Obtenir la quantité de réservation sous-utilisée et les coûts

Obtenez les données Coût amorti et appliquez le filtre _ChargeType_ _= UnusedReservation_. Vous obtenez la quantité de réservation inutilisée quotidienne et le coût. Vous pouvez filtrer les données pour une réservation ou un ordre de réservation à l’aide des champs _ReservationId_ et _ProductOrderId_, respectivement. Si une réservation a été utilisée à 100 %, l’enregistrement a une quantité égale à 0.

### <a name="amortize-reservation-costs"></a>Amortir les coûts des réservations

Obtenez les données Coût amorti et filtrez pour un ordre de réservation à l’aide de _ProductOrderID_ afin d’obtenir les coûts amortis quotidiens pour une réservation.

### <a name="chargeback-for-a-reservation"></a>Rétrofacturer une réservation

Vous pouvez rétrofacturer l’utilisation d’une réservation à d’autres organisations par abonnement, groupes de ressources ou étiquettes. Les données des coûts amortis fournissent une valeur monétaire de l’utilisation d’une réservation en fonction des types de données suivants :

- Ressources (par exemple, une machine virtuelle)
- Resource group
- Balises
- Abonnement

### <a name="get-the-blended-rate-for-chargeback"></a>Obtenir le taux combiné pour la rétrofacturation

Pour déterminer le taux combiné, obtenez les données des coûts amortis et agrégez le coût total. Pour les machines virtuelles, vous pouvez utiliser les informations MeterName ou ServiceType à partir des données JSON AdditionalInfo. Divisez le coût total par la quantité utilisée pour obtenir le taux combiné.

### <a name="audit-optimum-reservation-use-for-instance-size-flexibility"></a>Auditer l’utilisation optimale de la réservation pour la flexibilité de la taille d’instance

Multipliez la quantité par le _RINormalizationRatio_, à partir d’AdditionalInfo. Le résultat indique le nombre d’heures d’utilisation de la réservation appliquées à l’enregistrement de l’utilisation.

### <a name="determine-reservation-savings"></a>Déterminer les économies de réservation

Obtenez les données des coûts amortis et filtrez les données pour une instance réservée. Ensuite :

1. Obtenez les coûts de paiement à l’utilisation estimés. Multipliez la valeur _UnitPrice_ par les valeurs _Quantity_ pour obtenir les coûts de paiement à l’utilisation estimés, si la remise de réservation n’a pas été appliquée à l’utilisation.
2. Obtenez les coûts de réservation. Effectuez la somme des valeurs _Cost_ pour obtenir la valeur monétaire de ce que vous avez payé pour l’instance réservée. Le résultat inclut les coûts utilisés et non utilisés de la réservation.
3. Soustrayez les coûts de réservation des coûts de paiement à l’utilisation estimés pour obtenir les économies estimées.

Gardez à l’esprit que si votre réservation est sous-utilisée, l’entrée _UnusedReservation_ pour _ChargeType_ devient un facteur à prendre en compte. Lorsque vous disposez d’une réservation intégralement utilisée, les économies que vous réalisez sont maximales. Toute quantité _UnusedReservation_ réduit les économies.

## <a name="reservation-purchases-and-amortization-in-cost-analysis"></a>Achats de réservation et amortissement dans l’analyse des coûts

Les coûts des réservations sont disponibles dans l’[analyse des coûts](https://aka.ms/costanalysis). Par défaut, l’analyse des coûts montre le **Coût réel**, qui est ce qui apparaîtra sur votre facture. Pour voir les achats des réservations ventilés et associés aux ressources qui ont utilisé l’avantage, passez au **Coût amorti** :

![Exemple montrant où sélectionner le coût amorti dans l’analyse des coûts](./media/understand-reserved-instance-usage-ea/portal-cost-analysis-amortized-view.png)

Regroupez par type de coût pour voir une ventilation de l’utilisation, des achats et des remboursements, ou regroupez par réservation pour une ventilation de la réservation et des coûts à la demande. Rappelez-vous que les seuls coûts de réservation que vous voyez en examinant le coût réel sont des achats, mais que les coûts sont alloués aux ressources individuelles qui ont utilisé l’avantage quand vous regardez le coût amorti. Vous voyez aussi un nouveau type de coût, **UnusedReservation**, quand vous regardez le coût amorti.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
- [Prépayer des machines virtuelles avec des instances de machines virtuelles réservées Azure](../../virtual-machines/prepay-reserved-vm-instances.md)
- [Prépayer des ressources de calcul SQL Database avec une capacité réservée Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Gérer les réservations Azure](manage-reserved-vm-instance.md)
- [Comprendre comment la remise de réservation est appliquée](../manage/understand-vm-reservation-charges.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
- [Coûts des logiciels Windows non inclus dans les réservations](reserved-instance-windows-software-costs.md)