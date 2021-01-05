---
title: Optimiser les coûts de Azure Databricks avec un pré-achat
description: Découvrez comment pré-payer Azure Databricks avec une capacité réservée pour économiser de l’argent.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: b7d7380114046fbeaaf4b6eb2660997d64168715
ms.sourcegitcommit: 5dbea4631b46d9dde345f14a9b601d980df84897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91368564"
---
# <a name="optimize-azure-databricks-costs-with-a-pre-purchase"></a>Optimiser les coûts de Azure Databricks avec un pré-achat

Vous pouvez économiser sur les coûts de votre unité Azure Databricks (DBU) quand vous pré-achetez des unités de validation Azure Databricks (DBCU) pendant une ou trois années. Vous pouvez utiliser les DBCU pré-achetés à tout moment pendant la durée de l’achat. Contrairement aux machines virtuelles, les unités pré-achetées n’expirent pas sur une base horaire et vous pouvez les utilisez à tout moment pendant la durée de l’achat.

Toutes les Azure Databricks utilisent automatiquement les déductions des DBU pré-achetées. Vous n’avez pas besoin de redéployer ou d’affecter un plan pré-installé à vos espaces de travail Azure Databricks pour l’utilisation de DBU afin d’obtenir les remises de pré-achat.

La remise de pré-achat s’applique uniquement à l’utilisation de DBU. Les autres frais liés au calcul, au stockage et à la mise en réseau sont facturés séparément.

## <a name="determine-the-right-size-to-buy"></a>Déterminez la taille appropriée à acheter

Le pré-achat de Databricks s’applique à toutes les charges de travail et les niveaux Databricks. Vous pouvez considérer le pré-achat comme un pool d’unités de validation Databricks pré-payé. L’utilisation est déduite du pool, quelle que soit la charge de travail ou le niveau. L’utilisation est déduite dans le rapport suivant :

| **Charge de travail** | **Rapport d’application DBU - niveau Standard** | **Rapport d’application DBU : niveau Premium** |
| --- | --- | --- |
| Analytique des données | 0.4 | 0,55 |
| Engineering données | 0.15 | 0.30 |
| Engineering données allégé | 0,07 | 0,22 |

Par exemple, lorsqu’une quantité d’analytique données (niveau Standard) est utilisée, les unités de validation Databricks pré-achetées sont déduites de 0,4 unités.

Avant de procéder à l’achat, calculez la quantité totale de DBU utilisée pour différentes charges de travail et niveaux. Utilisez les rapports précédents pour normaliser à DBCU, puis exécutez une projection de l’utilisation totale au cours d’une ou de trois années.

## <a name="purchase-databricks-commit-units"></a>Acheter des unités de validation Databricks

Vous pouvez acheter des plans Databricks dans le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22Databricks%22%7D). Pour acheter une capacité de réserve, vous devez avoir le rôle de propriétaire pour au moins un abonnement d’entreprise.

- Vous devez avoir un rôle de propriétaire pour au moins un Contrat Entreprise (références de l’offre : MS-AZR-0017P or MS-AZR-0148P) ou Microsoft Customer Agreement (MCA) ou un abonnement individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P).
- Pour les abonnements EA, l’option Ajouter des instances réservées doit être activée dans le portail EA. Ou, si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
- Pour les abonnements Entreprise, **Add Reserved Instances** (Ajouter des instances réservées) doit être activé dans le [portal EA](https://ea.azure.com/). Si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.

**Pour acheter :**

1. Accédez au [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22Databricks%22%7D).
1. Sélectionnez un abonnement. Utilisez la liste **Abonnement** pour choisir l’abonnement à utiliser pour payer la capacité réservée. Les coûts initiaux de la capacité réservée sont facturés conformément au mode de paiement défini sur l’abonnement. Les frais sont déduits du solde de l’engagement financier de l’inscription, ou facturés comme un dépassement.
1. Sélectionnez une étendue. Utilisez la liste **Étendue** pour choisir une étendue d’abonnement :
    - **Étendue de groupe de ressources unique** : applique la remise de réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.
    - **Étendue d’abonnement unique** : applique la remise de réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.
    - **Étendue partagée** : applique la remise de réservation aux ressources correspondantes dans les abonnements éligibles inclus dans le contexte de facturation. Pour les clients Contrat Entreprise, le contexte de facturation correspond à l’inscription.
1. Sélectionnez le nombre d’unités de validation Azure Databricks que vous souhaitez acheter et terminez l’achat.


![Exemple d’achat Azure Databricks dans le portail Azure](./media/prepay-databricks-reserved-capacity/data-bricks-pre-purchase.png)

## <a name="change-scope-and-ownership"></a>Modifiez l’étendue et la propriété

Vous pouvez apporter les modifications suivantes à une réservation après achat :

- Mettez à jour l’étendue de la réservation
- Contrôle d’accès en fonction du rôle Azure (Azure RBAC)

Vous ne pouvez pas fractionner ou fusionner l’unité de validation Databricks avant l’achat. Pour plus d’informations sur la gestion des réservations, consultez [Gérer les réservations après l’achat](manage-reserved-vm-instance.md).

## <a name="cancellations-and-exchanges"></a>Annulations et échanges

Les annulations et les échanges ne sont pas pris en charge pour les plans de pré-achat Databricks. Tous les achats sont finaux.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
  - [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
  - [Comprendre comment une remise DBCU Azure Databricks est appliquée avant l’achat](reservation-discount-databricks.md)
  - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
