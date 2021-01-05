---
title: Échanges et remboursements en libre-service de réservations Azure
description: Découvrez comment vous pouvez échanger des réservations Azure ou en obtenir le remboursement. Vous devez avoir un accès propriétaire à l’ordre de réservation pour échanger ou rembourser des réservations.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 12/15/2020
ms.author: banders
ms.openlocfilehash: b0f17149eb646b6108dc6a81922e6e5b00f143d6
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560540"
---
# <a name="self-service-exchanges-and-refunds-for-azure-reservations"></a>Échanges et remboursements en libre-service de réservations Azure

Les réservations Azure offrent la flexibilité nécessaire pour répondre à vos besoins en constante évolution. Vous pouvez échanger une réservation pour une autre réservation du même type. Par exemple, vous pouvez échanger une réservation de machine virtuelle pour acheter une autre réservation pour une autre région ou taille de machine virtuelle. De même, une réservation de base de données SQL PaaS peut être échangée en vue d’acheter une autre réservation pour n’importe quel type de base de données SQL PaaS ou région. Vous pouvez également rembourser des réservations, mais la somme totale de tous les engagements de réservation annulés dans votre étendue de facturation (par exemple, Contrat Entreprise, Contrat client Microsoft et Contrat Partenaire Microsoft) ne peut pas dépasser 50 000 USD dans une période de 12 mois. La capacité réservée Azure Databricks, la solution Azure VMware par réservation CloudSimple, la réservation de shift ouvert Azure Red Hat, les offres Red Hat et les offres SUSE Linux ne sont pas éligibles aux remboursements.

L’échange en libre service et la fonctionnalité d’annulation n’est pas disponible pour les clients Contrat Entreprise de l’administration américaine. Les autres types d’abonnements US Government, notamment les abonnements avec paiement à l’utilisation et les abonnements de fournisseur de solutions cloud sont pris en charge.

> [!NOTE]
> - **Vous devez disposer d’un accès propriétaire à l’ordre de réservation pour échanger une réservation existante ou en obtenir le remboursement.** . Vous pouvez [ajouter ou changer les utilisateurs qui peuvent gérer une réservation](./manage-reserved-vm-instance.md#who-can-manage-a-reservation-by-default).
> - Microsoft ne facture actuellement aucun frais de résiliation anticipée pour les remboursements de réservations. Il est possible que nous facturions les frais pour les remboursements effectués à l’avenir. Nous n’avons actuellement aucune date concernant l’activation des frais.

## <a name="how-to-exchange-or-refund-an-existing-reservation"></a>Comment échanger ou rembourser une réservation existante

Vous pouvez échanger votre réservation à partir du [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).

1. Sélectionnez les réservations dont vous souhaitez obtenir le remboursement et sélectionnez **Échanger**.  
    [![Exemple d’image illustrant des réservations à retourner](./media/exchange-and-refund-azure-reservations/exchange-refund-return.png)](./media/exchange-and-refund-azure-reservations/exchange-refund-return.png#lightbox)
1. Sélectionnez le produit de machine virtuelle que vous voulez acheter et tapez une quantité. Assurez-vous que le nouveau total d’achat est supérieur au total du retour. [Déterminez la bonne taille avant d’effectuer l’achat](../../virtual-machines/prepay-reserved-vm-instances.md#determine-the-right-vm-size-before-you-buy).  
    [![Exemple d’image illustrant le produit de machine virtuelle à acheter avec un échange](./media/exchange-and-refund-azure-reservations/exchange-refund-select-purchase.png)](./media/exchange-and-refund-azure-reservations/exchange-refund-select-purchase.png#lightbox)
1. Vérifiez et finalisez la transaction.  
    [![Exemple d’image illustrant le produit de machine virtuelle à acheter avec un échange, avec finalisation du retour](./media/exchange-and-refund-azure-reservations/exchange-refund-confirm-exchange.png)](./media/exchange-and-refund-azure-reservations/exchange-refund-confirm-exchange.png#lightbox)

Pour obtenir le remboursement d’une réservation, accédez à **Détails de la réservation** et sélectionnez **Remboursement**.

## <a name="exchange-non-premium-storage-for-premium-storage"></a>Échanger un stockage non Premium contre un stockage Premium

Vous pouvez échanger une réservation achetée pour une taille de machine virtuelle qui ne prend pas en charge le stockage Premium au profit d’une taille de machine virtuelle correspondante qui assure cette prise en charge. Par exemple, une machine virtuelle _F1_ contre une machine virtuelle _F1s_. Pour effectuer l’échange, accédez à Détails de la réservation, puis sélectionnez **Échanger**. L’échange ne réinitialise pas la période de l’instance réservée et ne crée pas de transaction. 

## <a name="how-transactions-are-processed"></a>Mode de traitement des transactions

Tout d’abord, Microsoft annule la réservation existante et rembourse le montant de cette réservation au prorata. S’il existe un échange, le nouvel achat est traité. Microsoft traite les remboursements à l’aide d’une des méthodes suivantes, selon vos type de compte et mode de paiement :

### <a name="enterprise-agreement-customers"></a>Clients titulaires d’un contrat entreprise

Pour les échanges et les remboursements, l’argent est ajouté à l’engagement financier éventuellement utilisé pour l’achat d’origine. Si la période de l’engagement financier au cours de laquelle la réservation a été achetée n’est plus active, le crédit est ajouté à la période actuelle de l’engagement financier rattaché à votre contrat entreprise. Le crédit est valable 90 jours à compter de la date du remboursement. Tout crédit inutilisé expire au bout de 90 jours.

Si l’achat d’origine a été effectué en tant que dépassement, la facture d’origine sur laquelle la réservation a été achetée et toutes les factures ultérieures sont rouvertes et réajustées. Microsoft émet une note de crédit pour les remboursements.

### <a name="pay-as-you-go-invoice-payments-and-csp-program"></a>Paiements des factures à l’utilisation et programme Fournisseur de solutions cloud

La facture d’achat de réservation d’origine est annulée, puis une facture est créée pour le remboursement. Pour les échanges, la nouvelle facture montre le remboursement et le nouvel achat. Le montant du remboursement est ajusté par rapport à l’achat. Dans le cas d’un simple remboursement d’une réservation, Microsoft conserve le montant au prorata, en vue de son ajustement par rapport à un achat de réservation futur. Si vous avez acheté une réservation aux tarifs du paiement à l’utilisation et que vous passez ensuite à un abonnement de fournisseur de solutions cloud, la réservation peut être retournée et rachetée sans pénalité.

### <a name="pay-as-you-go-credit-card-customers"></a>Clients qui paient à l’utilisation au moyen d’une carte de crédit

La facture d’origine est annulée, puis une facture est créée. L’argent est crédité sur la carte de crédit qui a été utilisée pour l’achat d’origine. Si vous avez changé de carte, [contactez le support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="cancel-exchange-and-refund-policies"></a>Stratégies d’annulation, d’échange et de remboursement

Azure dispose des stratégies suivantes pour les annulations, les échanges et les remboursements.

**Stratégies d’échange**

- Vous pouvez retourner plusieurs réservations existantes pour acheter une seule nouvelle réservation du même type. Vous ne pouvez pas échanger une réservation d’un type contre une réservation d’un autre type. Par exemple, vous ne pouvez pas retourner une réservation de machine virtuelle pour acheter une réservation SQL. Vous pouvez modifier une propriété d’une réservation telle que la famille, la série, la version, la référence SKU, la région, la quantité et la période avec un échange.
- Seuls les propriétaires de réservation peuvent traiter un échange. [Découvrez comment ajouter ou changer les utilisateurs qui peuvent gérer une réservation](manage-reserved-vm-instance.md#who-can-manage-a-reservation-by-default).
- Un échange est traité comme un remboursement et un rachat : différentes transactions sont créées pour l’annulation et le nouvel achat de réservation. Le montant de la réservation au prorata est remboursé pour les réservations qui sont échangées. Vous êtes facturé entièrement pour le nouvel achat. Le montant de la réservation au prorata est la valeur résiduelle au prorata quotidienne de la réservation qui fait l’objet d’un retour.
- Vous pouvez échanger une réservation ou en obtenir le remboursement même si le contrat entreprise utilisé pour son achat a expiré et a été renouvelé sous la forme d’un nouveau contrat.
- L’engagement de durée de vie de la nouvelle réservation doit être égal ou supérieur à l’engagement restant de la réservation retournée. Exemple : Pour une réservation de trois ans à 100 $ par mois et échangée après le 18ème paiement, l’engagement de durée de vie de la nouvelle réservation doit être de 1 800 $ ou plus (payés mensuellement ou d’avance).
- La nouvelle réservation achetée dans le cadre d’un échange a une nouvelle période qui commence à la date de cet échange.
- Il n’y a aucune pénalité ou limite annuelle pour les échanges.

**Stratégies de remboursement**

- Nous ne facturons actuellement aucun frais de résiliation anticipée, mais des frais de résiliation anticipée de 12 % pourraient s’appliquer à l’avenir en cas d’annulation.
- L’engagement total annulé ne peut pas dépasser 50 000 USD dans une période de 12 mois pour un profil de facturation ou une inscription unique. Par exemple, pour une réservation d’une durée de trois ans à 100 USD par mois et qui est remboursée le 18ème mois, l’engagement annulé est de 1 800 USD. Après le remboursement, votre nouvelle limite disponible pour le remboursement est de 48 200 USD. 365 jours à compter du remboursement, la limite de 48 200 USD sera augmentée de 1 800 USD et votre nouvelle réserve sera de 50 000 USD. Toute autre annulation de réservation pour le profil de facturation ou l’inscription à un Contrat Entreprise épuisera cette même réserve, et la même logique de réapprovisionnement s’appliquera.
- Azure ne traite pas les remboursements qui dépassent la limite de 50 000 USD dans une période de 12 mois pour un profil de facturation ou une inscription à un Contrat Entreprise.
- Les remboursements sont calculés en fonction du prix le plus bas de votre prix d’achat ou du prix actuel de la réservation.
- Seuls les propriétaires d’ordres de réservation peuvent traiter un remboursement. [Découvrez comment ajouter ou changer les utilisateurs qui peuvent gérer une réservation](manage-reserved-vm-instance.md#who-can-manage-a-reservation-by-default).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment gérer une réservation, consultez [Gérer les réservations Azure](manage-reserved-vm-instance.md).
- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
    - [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
    - [Gérer les réservations dans Azure](manage-reserved-vm-instance.md)
    - [Comprendre comment la remise de réservation est appliquée](../manage/understand-vm-reservation-charges.md)
    - [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
    - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
    - [Coûts des logiciels Windows non inclus dans les réservations](reserved-instance-windows-software-costs.md)
    - [Réservations Azure dans le cadre du programme Fournisseur de solutions cloud](/partner-center/azure-reservations)