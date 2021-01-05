---
title: Tarifs du calcul réservé - Azure Database pour PostgreSQL - Hyperscale (Citus)
description: Prépayer les ressources de calcul Azure Database pour PostgreSQL - Hyperscale (Citus) avec une capacité de réserve.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 06/15/2020
ms.openlocfilehash: f662d7e51c49006b191778ef70740ef79173828c
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92487940"
---
# <a name="prepay-for-azure-database-for-postgresql---hyperscale-citus-compute-resources-with-reserved-capacity"></a>Prépayer les ressources de calcul Azure Database pour PostgreSQL - Hyperscale (Citus) avec une capacité de réserve

Faites des économies avec Azure Database pour PostgreSQL – Hyperscale (Citus) en utilisant le prépaiement de ressources de calcul au lieu d’appliquer le tarif du paiement à l’utilisation. Avec une capacité réservée Hyperscale (Citus) , vous prenez un engagement initial pour le groupe de serveurs Hyperscale (Citus), sur une période d’un ou de trois ans, afin de bénéficier d’une remise importante sur les coûts de calcul. Pour acheter une capacité réservée Hyperscale (Citus), vous devez spécifier la région Azure, la durée de réservation et la fréquence de facturation.

> [!IMPORTANT]
> Cette page concerne la capacité de réserve d’Azure Database pour PostgreSQL – Hyperscale (Citus). Pour plus d’informations sur la capacité réservée pour Azure Database pour PostgreSQL – Serveur unique, consultez [Acompte pour Azure Database pour PostgreSQL – Ressources de calcul à serveur unique avec une capacité réservée](./concept-reserved-pricing.md).

Vous n’avez pas besoin d’attribuer cette réservation à des groupes de serveurs Hyperscale (Citus) spécifiques. Un groupe de serveur Hyperscale (Citus) déjà en cours d’exécution ou plusieurs groupes récemment déployés, bénéficient automatiquement du tarif réservé. En achetant une réservation, vous payez d’avance les coûts de calcul pendant une durée d’un an ou de trois ans. Dès que vous achetez une réservation, les frais de calcul Hyperscale (Citus) qui correspondent aux attributs de la réservation ne sont plus facturés au tarif du paiement à l’utilisation. 

Une réservation ne couvre pas les frais de logiciels, de mise en réseau ou de stockage qui sont associés aux groupes de serveurs Hyperscale (Citus). À l’issue de la période de réservation, la remise sur facturation cesse de s’appliquer et les groupes de serveurs Hyperscale (Citus) sont facturés au tarif du paiement à l’utilisation. Les réservations ne sont pas renouvelées automatiquement. Pour plus d’informations sur les tarifs, consultez l’[offre de capacité de réserve Azure Database pour PostgreSQL - Hyperscale (Citus)](https://azure.microsoft.com/pricing/details/postgresql/hyperscale-citus/).

Vous pouvez acheter une capacité de réserve Hyperscale (Citus) sur le [portail Azure](https://portal.azure.com/). Payez la réservation [à l’avance ou par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md). Pour acheter une capacité réservée :

* Vous devez disposer du rôle Propriétaire sur au moins un abonnement Entreprise (EA) ou individuel avec des tarifs de paiement à l’utilisation.
* Pour les abonnements Accord Entreprise, **Ajouter des instances réservées** doit être activé dans le [portail EA Portal](https://ea.azure.com/). Ou alors, si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
* Pour ce qui est du programme Fournisseur de solutions Cloud (CSP), seuls les agents d’administration ou les agents commerciaux peuvent acheter une capacité réservée Hyperscale (Citus).

Pour plus d’informations sur les clients Accord Entreprise Entreprise et le Paiement à l’utilisation pour l’achat de réservations, consultez :
- [Comprendre l’utilisation d’une réservation Azure pour votre Accord Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
- [Comprendre l’utilisation d’une réservation Azure pour votre abonnement avec Paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md)

## <a name="determine-the-right-server-group-size-before-purchase"></a>Déterminer la taille de groupe de serveurs adéquate avant l’achat

La taille de la réservation doit s'appuyer sur le nombre total de calculs utilisés par le coordinateur et les nœuds worker existants ou bientôt déployés dans les groupes de serveurs Hyperscale (Citus) au sein d’une région spécifique.

Supposons par exemple que vous exécutiez un groupe de serveurs Hyperscale (Citus) avec un coordinateur 16 vCore et trois nœuds worker 8 vCore. Admettons également que vous prévoyez de déployer au cours du mois prochain un groupe de serveur Hyperscale (Citus) supplémentaire avec un coordinateur 32 vCore et deux nœuds worker 4 vCore. Imaginons aussi que ces ressources vous seront nécessaires pendant au moins un an.

Dans ce cas, vous devriez acheter une réservation d’un an pour :

* Total 16 vCores + 32 vCores = 48 vCores pour les nœuds coordinateurs
* Total 3 nœuds x 8 vCores + 2 nœuds x 4 vCores = 24 + 8 = 32 vCores pour les nœuds worker

## <a name="buy-azure-database-for-postgresql-reserved-capacity"></a>Acheter une capacité de réserve Azure Database pour PostgreSQL

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez **Tous les services** > **Réservations** .
1. Sélectionnez **Ajouter** . Dans le volet **Acheter des réservations** , sélectionnez **Azure Database pour PostgreSQL** pour acheter une nouvelle réservation pour vos bases de données PostgreSQL.
1. Sélectionnez le type de **calcul Hyperscale (Citus)** à acheter et cliquez sur le bouton **Sélectionner** .
1. Vérifiez la quantité pour le type de calcul sélectionné depuis l’onglet **Produits** .
1. Passez à l’onglet **Acheter + Vérifier** pour finaliser votre achat.

Le tableau suivant décrit les champs requis.

| Champ        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Abonnement | L’abonnement utilisé pour payer la réservation de capacité réservée Azure Database pour PostgreSQL. Les coûts initiaux de la réservation de capacité réservée Azure Database pour PostgreSQL sont facturés selon le mode de paiement défini sur l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P ou MS-AZR-0148p) ou un accord individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P). Pour un abonnement Accord Entreprise, les frais sont déduits du solde de l’engagement financier de l’inscription, ou facturés comme un dépassement. Pour un abonnement individuel avec paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture sur l’abonnement.                                                                                  |
| Étendue        | L’étendue de la réservation vCore peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez **Partagé** , la remise de réservation vCore est appliquée aux groupes de serveurs Hyperscale (Citus) en cours d’exécution dans tous les abonnements de votre contexte de facturation. Pour les clients Accord Entreprise, l’étendue partagée correspond à l'inscription et inclut tous les abonnements compris dans l’inscription. Pour les clients Paiement à l’utilisation, l’étendue partagée correspond à tous les abonnements Paiement à l’utilisation créés par l’administrateur de compte. Si vous sélectionnez **Abonnement unique** , la remise de réservation vCore est appliquée aux groupes de serveurs Hyperscale (Citus) inclus dans cet abonnement. Si vous sélectionnez **Groupe de ressources unique** , la remise de réservation est appliquée aux groupes de serveurs Hyperscale (Citus) inclus dans l’abonnement sélectionné et dans le groupe de ressources sélectionné dans cet abonnement. |
| Région       | La région Azure couverte par la réservation de capacité réservée Azure Database pour PostgreSQL – Hyperscale (Citus).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Terme         | Une année ou trois ans.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Quantité     | Quantité de ressources de calcul achetée au sein de la réservation de capacité réservée Hyperscale (Citus). En particulier, le nombre de coordinateurs ou de nœuds worker vCores dans la région Azure sélectionnée qui sont réservés et qui bénéficient de la remise de facturation. Par exemple, si vous exécutez (ou prévoyez d’exécuter) des groupes de serveurs Hyperscale (Citus) avec une capacité de calcul totale de 64 vCores de nœud coordinateur et 32 vCores de nœud worker dans la région Est des États-Unis, vous devez spécifier une quantité de 64 et de 32 respectivement pour les nœuds coordinateurs et les nœuds worker afin de maximiser l’avantage pour tous les serveurs.                                                                                                                                                                                                                                                     |



## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="vcore-size-flexibility"></a>Flexibilité de la taille vCore

La flexibilité de la taille vCore vous permet de mettre à l’échelle les nœuds coordinateurs et les nœuds worker au sein d’une région, sans perdre le bénéfice de la capacité réservée.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

La remise de réservation vCore est appliquée automatiquement au nombre de groupes de serveurs Hyperscale (Citus) qui correspondent à l’étendue et aux attributs de la réservation de capacité réservée Azure Database pour PostgreSQL. Vous pouvez mettre à jour l’étendue de la réservation de capacité réservée Azure Database pour PostgreSQL – Hyperscale (Citus) par le biais du Portail Azure, de PowerShell, d’Azure CLI ou de l’API.

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

* [Qu’est-ce qu’une réservation Azure ?](../cost-management-billing/reservations/save-compute-costs-reservations.md)
* [Gérer les réservations Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md)
* [Comprendre la remise sur réservation Azure](../cost-management-billing/reservations/understand-reservation-charges.md)
* [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reservation-charges-postgresql.md)
* [Comprendre l’utilisation d’une réservation pour votre abonnement Accord Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
* [Réservations Azure dans le cadre du programme Fournisseur de solutions Cloud de l’Espace partenaires](/partner-center/azure-reservations)