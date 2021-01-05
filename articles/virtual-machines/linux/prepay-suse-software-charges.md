---
title: Prépayer les abonnements logiciels – Réservations Azure
description: Découvrez comment prépayer vos abonnements logiciels pour faire des économies par rapport au paiement à l’utilisation.
author: bandersmsft
manager: yashesvi
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: banders
ms.openlocfilehash: a527b75d376f05ab6190187b7a03d6da775055ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81759288"
---
# <a name="prepay-for-azure-software-plans"></a>Prépayer les abonnements logiciels Azure

Quand vous prépayez votre utilisation de logiciels SUSE et RedHat dans Azure, vous pouvez faire des économies par rapport au paiement à l’utilisation. Les remises s’appliquent uniquement aux compteurs SUSE et RedHat et non à l’utilisation de machines virtuelles. Vous pouvez acheter des réservations de machines virtuelles séparément pour réaliser des économies supplémentaires.

Vous pouvez acheter des abonnements logiciels SUSE et RedHat sur le portail Azure. Pour acheter un abonnement logiciel :

- Vous devez disposer du rôle Propriétaire sur au moins un abonnement Entreprise ou individuel avec des tarifs de paiement à l’utilisation.
- Pour les abonnements Entreprise, l’option **Ajouter des instances réservées** doit être activée sur le [portail EA](https://ea.azure.com/). Si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
- Pour le programme Fournisseur de solutions Cloud, seuls les administrateurs et les commerciaux peuvent acheter des abonnements logiciels.

## <a name="buy-a-software-plan"></a>Acheter un abonnement logiciel

1. Connectez-vous au portail Azure et accédez à [Réservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).
2. Cliquez sur **Ajouter**, puis sélectionnez l’abonnement logiciel que vous voulez acheter.
Renseignez les champs obligatoires. Toutes les machines virtuelles SUSE Linux ou RedHat qui correspondent aux attributs de ce que vous achetez bénéficient de la remise. Le nombre de déploiements qui peuvent bénéficier de la remise dépend de l’étendue et de la quantité sélectionnées.
3. Sélectionnez un abonnement. Cela détermine ce que vous payez.
Les coûts initiaux de la réservation sont facturés sur le mode de paiement de l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P ou MS-AZR - 0148p) ou un accord individuel avec paiement à l’utilisation numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P).
    - Pour un abonnement Entreprise, les frais sont déduits du solde d’engagement monétaire de l’inscription ou facturés comme un dépassement.
    - Pour un abonnement individuel avec paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture.
4. Sélectionnez une étendue. L’étendue peut couvrir un ou plusieurs abonnements (étendue partagée).
    - Abonnement unique : la remise sur l’abonnement est appliquée à l’utilisation correspondante de cet abonnement.
    - Partagé : la remise sur l’abonnement est appliquée aux instances correspondantes de tout abonnement compris dans votre contexte de facturation. Pour les clients Entreprise, le contexte de facturation correspond à l’inscription et comprend tous les abonnements compris dans l’inscription. Pour le forfait individuel avec des clients ayant choisi le paiement à l’utilisation, le contexte de facturation est tous les forfaits individuels avec abonnement à paiement à l’utilisation créés par l’administrateur de compte.
    - Groupe de ressources unique : applique la remise sur réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.
5. Sélectionnez un produit pour choisir la taille de machine virtuelle et le type d’image. La remise s’applique à la taille de machine virtuelle sélectionnée uniquement.
6. Sélectionnez une durée de 1 ou 3 ans.
7. Choisissez une quantité, c’est-à-dire le nombre d’instances de machine virtuelle prépayées pouvant bénéficier de la remise de facturation.
8. Ajoutez le produit au panier, vérifiez et achetez.

La remise sur réservation est automatiquement appliquée au compteur de logiciels pour lequel vous effectuez le prépaiement. Les frais de calcul de machines virtuelles ne sont pas couverts par l’abonnement. Vous pouvez acheter les réservations de machines virtuelles séparément.

## <a name="discount-applies-to-different-suse-vm-sizes"></a>La remise s’applique à différentes tailles de machine virtuelle SUSE

Comme pour les instances réservées de machine virtuelle, les abonnements logiciels SUSE Linux offrent une certaine flexibilité quant à la taille des instances. Votre remise s’applique même quand vous déployez une machine virtuelle dont la taille est différente de celle de l’abonnement SUSE que vous avez acheté. Pour plus d’informations, consultez [Comprendre comment la remise sur les abonnements logiciels est appliquée](../../cost-management-billing/reservations/understand-suse-reservation-charges.md).

## <a name="redhat-plan-discount"></a>Remise sur abonnement Red Hat

Les abonnements sont disponibles uniquement pour les machines virtuelles Red Hat Enterprise Linux. La remise ne s’applique pas aux machines virtuelles Red Hat Enterprise Linux SAP HANA ou RedHat Enterprise Linux SAP Business Apps.

Les remises sur abonnement RedHat s’appliquent uniquement à la taille de machine virtuelle que vous sélectionnez au moment de l’achat. Les abonnements RHEL ne peuvent pas être remboursés ou échangés après l’achat.


## <a name="cancellation-and-exchanges-not-allowed"></a>Les annulations et les échanges sont interdits

Vous ne pouvez pas annuler ou échanger un abonnement SUSE ou RedHat que vous avez acheté. Pour être sûr d’acheter l’abonnement logiciel qui vous convient, étudiez votre utilisation. Pour savoir comment déterminer ce que vous avez acheté, consultez [Comprendre comment la remise sur les abonnements logiciels est appliquée](../../cost-management-billing/reservations/understand-suse-reservation-charges.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment gérer une réservation, consultez [Gérer les réservations Azure](../../cost-management-billing/reservations/manage-reserved-vm-instance.md).

Pour en savoir plus, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Gérer les réservations dans Azure](../../cost-management-billing/reservations/manage-reserved-vm-instance.md)
- [Comprendre l’application des remises de réservation SUSE](../../cost-management-billing/reservations/understand-suse-reservation-charges.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](../../cost-management-billing/reservations/understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
