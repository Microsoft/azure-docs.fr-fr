---
title: Présentation de vos frais de service externe Azure
description: En savoir plus sur la facturation des frais des services externes, anciennement appelés Marketplace, dans Azure.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8da85ec5781ff9575cf380092fea9e41d6af8786
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686120"
---
# <a name="understand-your-azure-external-services-charges"></a>Comprendre vos frais de service externes Azure
Les services externes sont publiés par des éditeurs de logiciels tiers sur la Place de marché Azure. Par exemple, SendGrid est un service externe que vous pouvez acheter dans Azure, mais qui n’est pas publié par Microsoft. Certains produits Microsoft sont aussi vendus sur la Place de marché Azure.

## <a name="how-external-services-are-billed"></a>Facturation des services externes

- Si vous avez un Contrat Client Microsoft (MCA) ou un Contrat Partenaire Microsoft (MPA), vos services tiers sont facturés avec le reste de vos services Azure. [Vérifiez votre type de compte de facturation](#check-billing-account-type) pour voir si vous avez accès à un contrat MCA ou MPA.
- Si vous n’avez pas de contrat MCA ou MPA, vos services externes sont facturés séparément de vos services Azure. Vous recevrez deux factures pour chaque période de facturation : une pour les services Azure et l’autre pour les achats sur la Place de marché.
- Chaque service externe possède un modèle de facturation différent. Certains services sont facturés selon un mode de paiement à l’utilisation, tandis que d’autres utilisent un modèle de paiement mensuel fixe.
- Vous ne pouvez pas utiliser de crédits mensuels gratuits pour les services externes. Si vous utilisez un abonnement Azure incluant des [crédits gratuits](https://azure.microsoft.com/pricing/spending-limits/), ceux-ci ne peuvent pas s’appliquer aux montants des factures de services externes. Lorsque vous configurez un nouveau service externe ou une ressource, un avertissement s’affiche :

    ![Avertissement d’achat Marketplace](./media/understand-azure-marketplace-charges/credit-warning.png)

<!-- ## View external service spending and history in the Azure portal
You can view a list of the external services that are on each subscription within the [Azure portal](https://portal.azure.com/):

1. Sign in to the [Azure portal](https://portal.azure.com/) as the account administrator.
2. In the Hub menu, select **Subscriptions**.

    ![Select Subscriptions in the Hub menu](./media/understand-azure-marketplace-charges/sub-button.png)
3. In the **Subscriptions** blade, select the subscription that you want to view, and then select **External services**.

    ![Select a subscription in the billing blade](./media/understand-azure-marketplace-charges/select-sub-external-services.png)
4. You should see each of your external service orders, the publisher name, service tier you bought, name you gave the resource, and the current order status. To see past bills, select an external service.

    ![Select an external service](./media/understand-azure-marketplace-charges/external-service-blade2.png)
5. From here, you can view past bill amounts including the tax breakdown.

    ![View external services billing history](./media/understand-azure-marketplace-charges/billing-overview-blade.png) -->

## <a name="external-spending-for-ea-customers"></a>Dépenses externes pour les clients EA

Les clients de contrat Entreprise (EA) peuvent voir les dépenses du service externe et télécharger des rapports dans le portail EA. Consultez la page sur [Azure Marketplace pour les clients EA](https://ea.azure.com/helpdocs/azureMarketplace) pour commencer.

## <a name="view-and-download-invoices-for-external-services"></a>Voir et télécharger les factures pour les services externes

Si vous avez un Contrat Client Microsoft (MCA) ou un Contrat Partenaire Microsoft (MPA), vos services tiers sont facturés avec le reste de vos services Azure sur une même facture. [Vérifiez votre type de compte de facturation](#check-billing-account-type) pour voir si vous avez accès à un contrat MCA ou MPA. Si c’est le cas, consultez [Afficher et télécharger les factures dans le portail Azure](download-azure-invoice.md) pour voir les frais de vos fournisseurs tiers.

Si vous n’avez pas de contrat MCA ou MPA, les frais de vos fournisseurs tiers sont facturés de manière séparée. 

Les frais relatifs à la Place de marché Azure sont affichés dans votre devise locale.

Vous pouvez afficher et télécharger vos factures associées à la Place de marché Azure à partir du portail Microsoft Azure, en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez **Gestion des coûts + facturation**.
1. Dans le menu de gauche, sélectionnez **Factures**.
1. Dans la liste déroulante des abonnements, sélectionnez l’abonnement associé à vos services Place de marché.
1. Examinez la colonne **Type** dans la liste des factures. Si une facture porte sur un service Place de marché, le type est **Place de marché et réservations Azure**. 

    ![capture d’écran du type Place de marché Azure dans la grille des factures](./media/understand-azure-marketplace-charges/marketplace-type-twd.png)

1. Pour filtrer par type de façon à afficher uniquement les factures Place de marché et réservations Azure, sélectionnez le filtre **Type**. Sélectionnez ensuite **Place de marché et réservations Azure** dans la liste déroulante.

    ![capture d’écran du filtre Type sélectionné avec Place de marché et réservation Azure sélectionné dans la liste déroulante](./media/understand-azure-marketplace-charges/type-filter.png)

1. Sélectionnez l’icône de téléchargement à droite de la facture que vous souhaitez télécharger.

    ![capture d’écran qui montre l’icône de téléchargement sélectionnée pour la facture](./media/understand-azure-marketplace-charges/download-icon-marketplace.png)

1. Sous **Facture**, sélectionnez le bouton **Télécharger** de couleur bleue.

    ![capture d’écran qui montre le bouton de téléchargement de la facture dans le volet contextuel](./media/understand-azure-marketplace-charges/invoice-download-marketplace.png)

## <a name="pay-for-external-services-in-the-azure-portal"></a>Payer pour des services externes sur le portail Azure

Si vous avez un Contrat Client Microsoft (MCA) ou un Contrat Partenaire Microsoft (MPA), vos services tiers sont facturés avec le reste de vos services Azure. [Vérifiez votre type de compte de facturation](#check-billing-account-type) pour voir si vous avez accès à un contrat MCA ou MPA. Si c’est le cas, vous pouvez régler la facture entière sur le portail Azure en suivant les étapes indiquées dans [Payer votre facture Microsoft Azure](pay-bill.md).

Si vous n’avez pas de contrat MCA ou MPA, vous pouvez régler vos factures de la Place de marché sur le portail Azure en suivant ces étapes :

1. Suivez les étapes de la section précédente, [Voir et télécharger les factures pour les services externes](#view-and-download-invoices-for-external-services), pour trouver vos factures de la Place de marché.
1. Sélectionnez le lien de couleur bleue **Payer maintenant** pour la facture que vous voulez régler.

    ![capture d’écran avec le lien Payer maintenant sélectionné dans la grille des factures](./media/understand-azure-marketplace-charges/pay-now-twd.png)

    >[!NOTE]
    > Le lien **Payer maintenant** ne s’affiche que si le type de votre facture est **Place de marché et réservations Azure** et si le statut de paiement de la facture est Échéance ou En retard.

1. Dans la nouvelle page, cliquez sur le lien de couleur bleue **Sélectionner le mode de paiement**.

    ![capture d’écran avec le lien de sélection du mode de paiement sélectionné](./media/understand-azure-marketplace-charges/select-payment-method-pay-now-twd.png)

1. Après avoir sélectionné un mode de paiement, cliquez sur le lien de couleur bleue **Payer maintenant** en bas à gauche de la page.
    ![capture d’écran avec le bouton Payer maintenant sélectionné](./media/understand-azure-marketplace-charges/pay-now-button-twd.png)

## <a name="change-default-payment-for-external-services"></a>Changer de mode de paiement par défaut pour les services externes

Lorsque vous achetez un service externe, vous choisissez un abonnement Azure pour la ressource. Le mode de paiement de l’abonnement Azure sélectionné devient le mode de paiement du service externe. Pour modifier le mode de paiement d’un service externe, vous devez [changer le mode de paiement de l’abonnement Azure](../manage/change-credit-card.md) lié à ce service externe. Vous pouvez identifier l’abonnement auquel votre commande de service externe est liée en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Cliquez sur l’option **Toutes les ressources** figurant dans le menu de navigation gauche.
     ![Capture d’écran de l’élément de menu Toutes les ressources](./media/understand-azure-marketplace-charges/all-resources.png)
1. Recherchez votre service externe.
1. Dans la colonne **Abonnement**, recherchez le nom de l’abonnement.
    ![Capture d’écran du nom d’abonnement pour la ressource](./media/understand-azure-marketplace-charges/sub-selected.png)
1. Cliquez sur le nom de l’abonnement et [mettez à jour le mode de paiement actif](../manage/change-credit-card.md).

## <a name="cancel-an-external-service-order"></a>Annuler une commande de service externe

Si vous souhaitez annuler votre commande de service externe, supprimez la ressource dans le [portail Azure](https://portal.azure.com).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Cliquez sur l’option **Toutes les ressources** figurant dans le menu de navigation gauche.
    ![Capture d’écran de l’élément de menu Toutes les ressources](./media/understand-azure-marketplace-charges/all-resources.png)
1. Recherchez votre service externe.
1. Cochez la case en regard de la ressource à supprimer.
1. Sélectionnez **Supprimer** dans la barre de commandes.
    ![Capture d’écran du bouton Supprimer](./media/understand-azure-marketplace-charges/delete-button.png)
1. Cliquez sur *Oui* dans le panneau de confirmation.
    ![Supprimer la ressource](./media/understand-azure-marketplace-charges/delete-resource.PNG)
1. Cliquez sur **Supprimer**.

## <a name="check-billing-account-type"></a>Vérifier le type de compte de facturation
[!INCLUDE [billing-check-account-type](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes
- [Démarrer l’analyse des coûts](../costs/quick-acm-cost-analysis.md)