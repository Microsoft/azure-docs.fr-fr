---
title: Capacité réservée dans Azure Cosmos DB pour optimiser les coûts
description: Découvrez comment acheter une capacité réservée Azure Cosmos DB pour réduire vos coûts de calcul.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/18/2020
ms.author: tisande
ms.reviewer: sngun
ms.openlocfilehash: 27725b1a3dd6059010ce67977c39891a012c037e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995781"
---
# <a name="optimize-cost-with-reserved-capacity-in-azure-cosmos-db"></a>Optimiser les coûts avec une capacité réservée dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

La capacité réservée Azure Cosmos DB vous permet de faire des économies en vous engageant à réserver des ressources Azure Cosmos DB sur un an ou trois ans. Avec la capacité réservée Azure Cosmos DB, vous pouvez obtenir une remise sur le débit provisionné pour les ressources Cosmos DB. Les bases de données et les conteneurs (tables, collections et graphes) sont des exemples de ressources.

La capacité réservée Azure Cosmos DB peut réduire sensiblement vos coûts Cosmos DB, jusqu’à 65 % sur les prix habituels avec un engagement d’un ou de trois ans. La capacité réservée permet de bénéficier d’une remise, sans impacter l’état d’exécution de vos ressources Azure Cosmos DB.

La capacité réservée Azure Cosmos DB couvre le débit provisionné pour vos ressources. Elle ne couvre pas les frais de stockage et de réseau. Dès que vous achetez une réservation, les frais de débit qui correspondent aux attributs de la réservation ne sont plus facturés au tarif du paiement à l’utilisation. Pour plus d’informations sur les réservations, consultez l’article [Réservations Azure](../cost-management-billing/reservations/save-compute-costs-reservations.md).

Vous pouvez acheter de la capacité réservée Azure Cosmos DB sur le [portail Azure](https://portal.azure.com). Payez la réservation [à l’avance ou par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md). Pour acheter une capacité réservée :

* Vous devez disposer du rôle Propriétaire sur au moins un abonnement Entreprise ou individuel avec des tarifs de paiement à l’utilisation.  
* Pour les abonnements Entreprise, **Add Reserved Instances** (Ajouter des instances réservées) doit être activé dans le [portal EA](https://ea.azure.com). Si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
* Pour le programme Fournisseur de solutions cloud, seuls les agents d’administration ou les agents commerciaux peuvent acheter une capacité réservée Azure Cosmos DB.

## <a name="determine-the-required-throughput-before-purchase"></a>Déterminer le débit requis avant l’achat

La taille de l’achat de la capacité de réserve doit être basée sur la quantité totale de débit qui sera utilisée sur une base horaire par les ressources Azure Cosmos DB existantes ou sur le point d’être déployées. Par exemple : Achetez une capacité réservée de 30 000 RU/s s’il s’agit de votre modèle d’utilisation horaire habituel. Dans cet exemple, tout débit approvisionné au-delà de 30 000 RU/s sera facturé au tarif Paiement à l’utilisation. Si le débit approvisionné est inférieur à 30 000 RU/s en une heure, la capacité de réserve supplémentaire pour cette heure est gaspillée.

Nous calculons les recommandations d’achat en fonction de votre modèle d’utilisation horaire. L’utilisation au cours des 7, 30 et 60 derniers jours est analysée et l’achat de capacité de réserve qui optimise vos économies est recommandé. Vous pouvez afficher les tailles de réservation recommandées dans le Portail Azure en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).  

2. Sélectionnez **Tous les services** > **Réservations** > **Ajouter**.

3. À partir du volet **Acheter des réservations**, choisissez **Azure Cosmos DB**.

4. Sélectionnez l’onglet **Recommandé** pour afficher les réservations recommandées :

Vous pouvez filtrer les recommandations selon les attributs suivants :

- **Durée** (1 an ou 3 ans)
- **Périodicité de facturation** (mensuelle ou comptant)
- **Type de débit** (RU/s par rapport à RU/s en écriture multi-région)

En outre, vous pouvez choisir l’étendue des recommandations pour qu’elles se trouvent dans un seul groupe de ressources, dans un seul abonnement ou dans l’intégralité de votre inscription Azure. 

Voici un exemple de recommandation :

:::image type="content" source="./media/cosmos-db-reserved-capacity/reserved-capacity-recommendation.png" alt-text="Recommandations sur la capacité de réserve":::

Cette recommandation d’achat indique que, parmi les réservations d’une durée de 3 ans, une taille de réservation de 30 000 RU/s maximisera les économies. Dans ce cas, la recommandation est calculée sur la base des 30 derniers jours d’utilisation d’Azure Cosmos DB. Si ce client s’attend à ce que les 30 derniers jours d’utilisation d’Azure Cosmos DB soient représentatifs de son utilisation future, il maximisera ses économies en achetant une réservation de 30 000 RU/s.

## <a name="buy-azure-cosmos-db-reserved-capacity"></a>Acheter une capacité réservée Azure Cosmos DB

1. Connectez-vous au [portail Azure](https://portal.azure.com).  

2. Sélectionnez **Tous les services** > **Réservations** > **Ajouter**.  

3. À partir du volet **Acheter des réservations**, choisissez **Azure Cosmos DB** pour acheter une nouvelle réservation.  

4. Renseignez les champs obligatoires, comme décrit dans le tableau suivant :

   :::image type="content" source="./media/cosmos-db-reserved-capacity/fill-reserved-capacity-form.png" alt-text="Remplir le formulaire de réservation de capacité":::

   |Champ  |Description  |
   |---------|---------|
   |Étendue   |   Option qui contrôle le nombre d’abonnements pouvant bénéficier de la remise sur la facturation associé à la réservation. Elle contrôle également la manière dont la réservation est appliquée à des abonnements spécifiques. <br/><br/>  Si vous sélectionnez **Partagé**, la remise de réservation est appliquée aux instances Azure Cosmos DB qui sont exécutées dans tous les abonnements de votre contexte de facturation. Le contexte de facturation est basé sur la façon dont vous vous êtes inscrit dans Azure. Pour les clients Entreprise, l'étendue partagée correspond à l'inscription et inclut tous les abonnements compris dans l'inscription. Pour les clients avec paiement à l’utilisation, l’étendue partagée correspond à tous les abonnements individuels avec des tarifs de paiement à l’utilisation créés par l’administrateur de compte.  <br/><br/>  Si vous sélectionnez **Abonnement unique**, la remise de réservation est appliquée aux instances Azure Cosmos DB incluses dans l’abonnement sélectionné. <br/><br/> Si vous sélectionnez **Groupe de ressources unique**, la remise de réservation est appliquée aux instances Azure Cosmos DB incluses dans l’abonnement sélectionné et dans le groupe de ressources sélectionné dans cet abonnement. <br/><br/> Vous pouvez changer l’étendue de la réservation après l’achat de la capacité réservée.  |
   |Abonnement  |   Abonnement utilisé pour payer la capacité réservée Azure Cosmos DB. Les coûts sont facturés selon le mode de paiement défini sur l’abonnement sélectionné. L’abonnement doit être de l’un des types suivants : <br/><br/>  Contrat Entreprise (références de l’offre : MS-AZR-0017P ou MS-AZR-0148P) : Pour un abonnement Entreprise, les frais sont déduits du solde de l’engagement financier de l’inscription, ou facturés comme un dépassement. <br/><br/> Abonnement individuel avec tarifs de paiement à l’utilisation (références de l’offre : MS-AZR-0003P ou MS-AZR-0023P) : Pour un abonnement individuel avec tarifs de paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture, défini sur l’abonnement.    |
   | Groupe de ressources | Groupe de ressources auquel la remise de capacité réservée est appliquée. |
   |Terme  |   Une année ou trois ans.   |
   |Type de débit   |  Le débit est provisionné en unités de requête. Vous pouvez acheter une réservation pour le débit approvisionné pour les deux configurations (écritures dans une seule région et écritures dans plusieurs régions). Le type de débit propose deux valeurs à sélectionner : 100 écritures de RU/s par heure et 100 écritures multi-régions de RU/s par heure.|
   | Unités de capacité réservée| Quantité de débit que vous souhaitez réserver. Pour calculer cette valeur, déterminez d’abord le débit nécessaire pour toutes vos ressources Cosmos DB (par exemple, les bases de données ou les conteneurs) par région. Ensuite, multipliez cette quantité par le nombre de régions à associer à votre base de données Cosmos. Par exemple : Si vous avez cinq régions avec chacune 1 million d’unités de requête par seconde (RU/s), sélectionnez 5 millions de RU/s comme capacité réservée à acheter. |


5. Le prix requis pour acheter la capacité réservée est calculé une fois le formulaire rempli. La sortie indique également le pourcentage de remise que vous obtenez avec les options sélectionnées. Cliquez ensuite sur **Sélectionner**

6. Vérifiez la remise et le prix de la réservation dans le volet **Acheter des réservations**. Ce prix de réservation s’applique aux ressources Azure Cosmos DB avec débit provisionné dans toutes les régions.  

   :::image type="content" source="./media/cosmos-db-reserved-capacity/reserved-capacity-summary.png" alt-text="Résumé de la capacité réservée":::

7. Sélectionnez **Évaluer et acheter**, puis **Acheter maintenant**. Vous voyez la page suivante une fois l’achat correctement effectué :

Une fois achetée, une réservation est immédiatement appliquée à toutes les ressources Azure Cosmos DB existantes qui répondent à ses conditions. Si vous n’avez pas de ressources Azure Cosmos DB, la réservation s’applique quand vous déployez une nouvelle instance Cosmos DB qui satisfait aux conditions de la réservation. Dans les deux cas, la période de la réservation démarre une fois l’achat correctement effectué.

Quand votre réservation expire, vos instances Azure Cosmos DB continuent de s’exécuter et sont facturées aux prix habituels du paiement à l’utilisation.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="next-steps"></a>Étapes suivantes

La remise de réservation est appliquée automatiquement aux ressources Azure Cosmos DB qui correspondent à l’étendue et aux attributs de la réservation. Vous pouvez mettre à jour l’étendue de la réservation par le biais du portail Azure, de PowerShell, d’Azure CLI ou de l’API.

*  Pour découvrir comment les remises sur capacité réservée sont appliquées à Azure Cosmos DB, consultez [Comprendre la remise sur réservation Azure](../cost-management-billing/reservations/understand-cosmosdb-reservation-charges.md).

* Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

   * [Qu’est-ce qu’une réservation Azure ?](../cost-management-billing/reservations/save-compute-costs-reservations.md)  
   * [Gérer les réservations Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md)  
   * [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)  
   * [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md)
   * [Réservations Azure dans le cadre du programme Fournisseur de solutions cloud de l’Espace Partenaires](/partner-center/azure-reservations)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).