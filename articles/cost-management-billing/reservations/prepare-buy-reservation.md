---
title: Acheter une réservation Azure
description: Découvrez les points importants à prendre en compte avant l’achat d’une réservation Azure.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: b03db9632f5f47a7600c5b9037d16b1cff6ccf69
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96344247"
---
# <a name="buy-a-reservation"></a>Acheter une réservation

Les réservations Azure vous permettent d’économiser en vous engageant sur des plans d’un ou trois ans pour de nombreuses ressources Azure. Avant de vous engager dans l’achat d’une réservation, veillez à consulter les sections suivantes, qui vous permettront de préparer votre achat.

## <a name="who-can-buy-a-reservation"></a>Qui peut acheter une réservation ?

Pour acheter une réservation, vous devez détenir le rôle de propriétaire ou d’acheteur de réservation sur un abonnement Azure de type Entreprise (MS-AZR-0017P ou MS-AZR-0148P), Paiement à l’utilisation (MS-AZR-0003P ou MS-AZR-0023P) ou Contrat client Microsoft. Les fournisseurs de solutions cloud peuvent utiliser le portail Azure ou l’ [Espace partenaires](/partner-center/azure-reservations) pour acheter des réservations Azure.

Les clients Contrat Entreprise (EA) peuvent limiter les achats aux administrateurs EA en désactivant l'option **Ajouter des instances réservées** dans le portail EA. Les administrateurs EA doivent disposer d’un accès propriétaire ou acheteur de réservation sur au moins un abonnement EA pour pouvoir acheter une réservation. Cette option est particulièrement utile pour les entreprises souhaitant qu’une équipe centralisée achète des réservations.

Une remise de réservation s’applique uniquement aux ressources associées aux abonnements qui ont été achetés dans le cadre de plans Entreprise, CSP, Contrat Client Microsoft et individuels avec paiement à l’utilisation.

## <a name="scope-reservations"></a>Réservations d’étendue

Vous pouvez étendre une réservation à un groupe de ressources ou à un groupe d’abonnement. La définition de l’étendue d’une réservation sélectionne où les économies de la réservation s’appliquent. Quand vous limitez l’étendue de la réservation à un groupe de ressources, les remises sur réservation s’appliquent uniquement au groupe de ressources et non à l’ensemble de l’abonnement.

### <a name="reservation-scoping-options"></a>Options d’étendue de réservation

Vous disposez de trois options pour définir l’étendue d’une réservation, selon vos besoins :

- **Étendue de groupe de ressources unique** : applique la remise sur réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.
- **Étendue d’abonnement unique** : applique la remise sur réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.
- **Étendue partagée** : applique la remise de réservation aux ressources correspondantes dans les abonnements éligibles inclus dans le contexte de facturation.
    - Pour les clients Contrat Entreprise, le contexte de facturation correspond à l’inscription. L’étendue partagée de réservation inclut plusieurs locataires Active Directory dans une inscription.
    - Pour les clients avec un Contrat Client Microsoft, l’étendue de facturation correspond au profil de facturation.
    - Pour les abonnements individuels utilisant les tarifs du paiement à l’utilisation, l’étendue de facturation correspond à tous les abonnements éligibles créés par l’administrateur de compte.

Lors de l’application des remises de réservation sur votre utilisation, Azure traite la réservation dans l’ordre suivant :

1. Réservations limitées à un groupe de ressources
2. Réservations à étendue unique
3. Réservations à étendue partagée

Vous pouvez modifier l’étendue après l’achat d’une réservation. Pour ce faire, accédez à la réservation, cliquez sur **Configuration** et redéfinissez la réservation. La modification de l’étendue de la réservation n’est pas une transaction commerciale. Le terme de votre réservation n’est pas modifié. Pour plus d’informations sur la mise à jour de l’étendue, consultez [Mettre à jour la portée après avoir acheté une réservation](manage-reserved-vm-instance.md#change-the-reservation-scope).

![Exemple illustrant la modification d’une étendue de réservation](./media/prepare-buy-reservation/rescope-reservation-resource-group.png)

## <a name="discounted-subscription-and-offer-types"></a>Types d'abonnements et d'offres remisés

Les remises de réservation s’appliquent aux types d'abonnements et d'offres éligibles suivants.

- Contrat Entreprise (références de l’offre : MS-AZR-0017P ou MS-AZR-0148P)
- Abonnements avec Contrat Client Microsoft.
- Forfaits individuels avec paiement à l’utilisation (références de l’offre : MS-AZR-0003P ou MS-AZR-0023P)
- Abonnements CSP

Les ressources opérant dans le cadre d’un abonnement relevant d’autres types d’offres ne bénéficient d’aucune remise sur réservation.

## <a name="purchase-reservations"></a>Acheter des réservations

Vous pouvez acheter des réservations à partir du portail Azure, d’API, de PowerShell et de l’interface CLI. Quand vous êtes prêt à acheter une réservation, lisez les articles s’appliquant à votre situation :

- [App Service](prepay-app-service-isolated-stamp.md)
- [Cache Azure pour Redis](../../azure-cache-for-redis/cache-reserved-pricing.md)
- [Cosmos DB](../../cosmos-db/cosmos-db-reserved-capacity.md)
- [Databricks](prepay-databricks-reserved-capacity.md)
- [Explorateur de données](/azure/data-explorer/pricing-reserved-capacity)
- [Stockage sur disque](../../virtual-machines/disks-reserved-capacity.md)
- [Hôte dédié](../../virtual-machines/prepay-dedicated-hosts-reserved-instances.md)
- [Abonnements logiciels](../../virtual-machines/linux/prepay-suse-software-charges.md)
- [Stockage](../../storage/blobs/storage-blob-reserved-capacity.md)
- [Base de données SQL](../../azure-sql/database/reserved-capacity-overview.md)
- [Base de données Azure pour PostgreSQL](../../postgresql/concept-reserved-pricing.md)
- [Azure Database pour MySQL](../../mysql/concept-reserved-pricing.md)
- [Azure Database for MariaDB](../../mariadb/concept-reserved-pricing.md)
- [Azure Synapse Analytics](prepay-sql-data-warehouse-charges.md)
- [Machines virtuelles](../../virtual-machines/prepay-reserved-vm-instances.md)

## <a name="buy-reservations-with-monthly-payments"></a>Acheter des réservations avec paiements mensuels

Vous pouvez payer les réservations en effectuant des paiements mensuels. Contrairement à un achat initial dans lequel vous payez le montant total, l’option de paiement mensuel répartit uniformément le coût total de la réservation sur les différents mois de la période. Une même réservation avec paiement initial et avec paiements mensuels a le même coût total : vous ne payez pas de frais supplémentaires si vous optez pour le paiement mensuel.

Si la réservation est achetée à l’aide d’un Contrat client Microsoft, le montant de votre paiement mensuel peut varier en fonction du taux de change du mois en cours pour votre devise locale.

Les paiements mensuels ne sont disponibles pour ce qui suit : Databricks, réservations SUSE Linux, offres Red Hat et licences Azure Red Hat OpenShift.

### <a name="view-payments-made"></a>Afficher les paiements effectués

Vous pouvez afficher les paiements effectués à l’aide des API, des données d’utilisation et de l’analyse des coûts. Pour les réservations avec paiement mensuel, la fréquence a pour valeur **recurring** (périodique) dans les données d’utilisation et l’API de facturation des réservations. Pour les réservations avec paiement initial, la valeur est **onetime** (paiement en une seule fois).

L’analyse des coûts affiche les achats mensuels dans la vue par défaut. Appliquez le filtre **purchase** (achat) à **Charge type** (Type de frais) et **recurring** (périodique) pour **Frequency** (Fréquence) pour afficher tous les achats. Pour afficher uniquement les réservations, appliquez un filtre pour **Reservation**.

![Exemple montrant les coûts d’achat de réservation dans l’analyse des coûts](./media/prepare-buy-reservation/cost-analysis.png)

### <a name="exchange-and-refunds"></a>Échange et remboursements

Comme les autres réservations, vous pouvez demander le remboursement ou l’échange des réservations achetées avec facturation mensuelle. 

Quand vous échangez une réservation qui est payée mensuellement, le coût total de la durée de vie du nouvel achat doit être supérieur aux paiements restants qui sont annulés pour la réservation retournée. Il n’y a pas d’autres limites ou frais pour les échanges. Vous pouvez échanger une réservation acquittée avec paiement initial pour acheter une nouvelle réservation avec facturation mensuelle. Toutefois, la valeur de la durée de vie de la nouvelle réservation doit être supérieure à la valeur au prorata de la réservation retournée.

Si vous annulez une réservation payée mensuellement, les paiements futurs annulés sont comptabilisés dans la limite de remboursement de 50 000 USD.

Pour plus d’informations sur les échanges et les remboursements, consultez [Échanges et remboursements en libre-service de réservations Azure](exchange-and-refund-azure-reservations.md).

## <a name="reservation-notifications"></a>Notifications de réservation

Selon la façon dont vous payez votre abonnement Azure, des notifications de réservation sont envoyées par e-mail aux utilisateurs suivants dans votre organisation. Les notifications sont envoyées pour différents événements, notamment : 

- Purchase
- Expiration de la réservation à venir
- Expiry
- Renouvellement
- Annulation
- Étendue modifiée

Pour les clients avec des abonnements EA :

- Les notifications sont envoyées uniquement aux contacts de notification EA.
- Les utilisateurs ajoutés à une réservation à l’aide d’une autorisation Azure RBAC (IAM) ne reçoivent pas de notifications par e-mail.

Pour les clients qui ont des abonnements individuels :

- l’acheteur reçoit une notification d’achat.
- Au moment de l’achat, le propriétaire du compte de facturation de l’abonnement reçoit une notification d’achat.
- Le propriétaire du compte reçoit toutes les autres notifications.

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les réservations pour les ressources Azure](manage-reserved-vm-instance.md)