---
title: Consulter votre facture de Contrat Partenaire Microsoft - Azure
description: Découvrez comment passer en revue votre facture et votre utilisation des ressources, et vérifier les frais associés à votre facture de Contrat client Microsoft.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 9e5ecc67fe86afa15c66d175f0705818154588bf
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684352"
---
# <a name="tutorial-review-your-microsoft-partner-agreement-invoice"></a>Tutoriel : Passer en revue votre facture de Contrat Partenaire Microsoft

 Dans le compte de facturation d’un Contrat Partenaire Microsoft, une facture est générée chaque mois pour chaque profil de facturation. La facture inclut tous les frais facturés aux clients depuis le mois précédent. Pour comprendre les frais indiqués sur votre facture, vous pouvez analyser les transactions individuelles dans le portail Azure. Vous pouvez également consulter vos factures dans le Portail Azure et comparer les frais au fichier de détails d’utilisation.

Pour plus d’informations, consultez l’article qui explique [comment télécharger des factures à partir du portail Azure](download-azure-invoice.md).

Ce tutoriel s’applique uniquement aux partenaires Azure disposant d’un Contrat Partenaire Microsoft.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Passer en revue les transactions facturées dans le Portail Azure
> * Passez en revue les frais en attente pour estimer votre prochaine facture
> * Analyser vos frais d’utilisation Azure

## <a name="prerequisites"></a>Prérequis

Vous devez avoir accès à un compte de facturation associé à un Contrat Partenaire Microsoft.

Vous devez être inscrit à Azure depuis plus de 30 jours. Azure vous facture à la fin de votre période de facturation.

## <a name="sign-in-to-azure"></a>Connexion à Azure

- Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Vérifier l'accès à un contrat client Microsoft

Vérifiez le type de contrat pour déterminer si vous avez accès à un compte de facturation d’un Contrat Partenaire Microsoft.

Dans le Portail Azure, tapez *gestion des coûts + facturation* dans la zone de recherche, puis sélectionnez **Gestion des coûts + facturation**.

![Capture d’écran montrant la recherche dans le portail Azure](./media/review-partner-agreement-bill/billing-search-cost-management-billing.png)

Si vous avez accès à une seule étendue de facturation, sélectionnez **Propriétés** dans la partie gauche. Vous avez accès à un compte de facturation d’un Contrat Partenaire Microsoft si le type de compte de facturation est **Contrat Partenaire Microsoft**.

![Capture d’écran montrant la page de propriétés d’un Contrat Partenaire Microsoft](./media/review-partner-agreement-bill/billing-account-properties-partner-agreement.png)

Si vous avez accès à plusieurs étendues de facturation, vérifiez le type indiqué dans la colonne du compte de facturation. Vous avez accès à un compte de facturation d’un Contrat Partenaire Microsoft si le type de compte de facturation de l’une des étendues est **Contrat Partenaire Microsoft**.

![Capture d’écran montrant la page listant les comptes de facturation d’un Contrat Partenaire Microsoft](./media/review-partner-agreement-bill/mpa-in-the-list.png)

## <a name="review-invoiced-transactions-in-the-azure-portal"></a>Passer en revue les transactions facturées dans le Portail Azure

Dans Gestion des coûts + facturation, sélectionnez **Toutes les transactions** sur le côté gauche de la page. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation, un profil de facturation ou un client avant de sélectionner **Toutes les transactions**.

La page Toutes les transactions affiche les informations suivantes :

![Capture d’écran montrant la liste des transactions facturées](./media/review-partner-agreement-bill/all-transactions.png)

|Colonne  |Définition  |
|---------|---------|
|Date     | Date de transaction  |
|ID de facture     | L’identificateur de la facture où la transaction a été facturée. Si vous envoyez une demande de support, partagez l’ID avec le support Azure pour accélérer le traitement de votre demande |
|Type de transaction     |  Il existe différents types de transactions : des frais d’achats, d’annulation et d’utilisation  |
|Famille de produits     | La catégorie du produit, tel que le calcul pour les machines virtuelles ou la base de données pour Azure SQL Database|
|Référence SKU de produit     | Un code unique identifiant l’instance de votre produit |
|Montant     |  Le montant de la transaction      |
|Profil de facturation     | La transaction s’affiche sur cette facture du profil de facturation |

Recherchez un ID de facture pour filtrer les transactions de la facture.

## <a name="review-pending-charges-to-estimate-your-next-invoice"></a>Passez en revue les frais en attente pour estimer votre prochaine facture

Les frais sont estimés et restent en attente jusqu’à ce qu’ils soient facturés. Pour estimer votre prochaine facture, vous pouvez consulter les frais en attente sur votre profil de facturation associé à un Contrat Partenaire Microsoft dans le portail Azure. Les frais en attente sont des estimations et s’entendent hors taxes. Les frais réels figurant sur votre prochaine facture seront différents des frais en attente.

### <a name="view-pending-transactions"></a>Afficher les transactions en attente

Lorsque vous avez identifié les frais en attente, vous pouvez les comprendre en analysant les transactions individuelles qui forment ces frais. À ce stade, les frais d’utilisation en attente ne sont pas affichés sur la page Toutes les transactions. Vous pouvez les afficher sur la page Abonnements Azure.

Dans Gestion des coûts + facturation, sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation. Dans le compte de facturation, sélectionnez **Profils de facturation**, puis un profil de facturation.

Sur le côté gauche de la page, sélectionnez **Toutes les transactions**.

Recherchez *En attente*. Utilisez le filtre **Timespan** (Période) pour afficher les frais en attente pour le mois actuel ou le mois dernier.

<!-- ![Screenshot that shows the pending transactions list](./media/billing-mpa-understand-your-bill/mpa-billing-profile-pending-transactions.png) -->

### <a name="view-pending-charges-by-customer"></a>Consulter les frais en attente par client

Dans Gestion des coûts + facturation, sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation. Dans le compte de facturation, sélectionnez **Profils de facturation**, puis un profil de facturation.

Sélectionnez **Clients** sur le côté gauche de la page.

<!-- ![screenshot of billing profile customers list](./media/billing-mpa-understand-your-bill/mpa-billing-profile-customers.png) -->

La page Clients indique affiche les frais du mois actuel et du mois dernier pour chaque client associé au profil de facturation. Les frais du mois en cours sont des frais en attente. Ils seront facturés lors de la génération de la facture à la fin du mois. Si la facture du mois dernier n’est pas encore générée, les frais du mois dernier sont également en attente.

### <a name="view-pending-usage-charges"></a>Afficher les frais d’utilisation en attente

Dans Gestion des coûts + facturation, sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation. Dans le compte de facturation, sélectionnez **Profils de facturation**, puis un profil de facturation.

Sur le côté gauche de la page, sélectionnez **Abonnements Azure**. La page Abonnements Azure affiche les frais actuels et ceux du mois dernier pour chaque abonnement dans le profil de facturation. Les frais du mois en cours sont des frais en attente. Ils seront facturés lors de la génération de la facture à la fin du mois. Si la facture du mois dernier n’est pas encore générée, les frais du mois dernier sont également en attente.

<!--     ![Screenshot that shows the Azure subscriptions list for MPA billing profile](./media/billing-mpa-understand-your-bill/mpa-billing-profile-subscriptions-list.png) -->

## <a name="analyze-your-azure-usage-charges"></a>Analyser vos frais d’utilisation Azure

Utilisez le fichier CSV Utilisation et frais Azure pour analyser vos frais en fonction de votre utilisation. Vous pouvez filtrer le fichier Utilisation et frais Azure pour rapprocher les frais d’utilisation de chaque produit listé sur la facture PDF. Pour voir les frais d’utilisation détaillés d’un produit en particulier, filtrez la colonne **product** dans le fichier CSV Utilisation et frais Azure pour inclure uniquement le nom de ce produit.

Vous pouvez également filtrer la colonne **customerName** dans le fichier CSV Utilisation et frais Azure afin de voir les frais d’utilisation quotidienne facturés à chacun de vos clients. Si vous souhaitez voir les frais d’utilisation quotidienne par abonnement Azure, filtrez la colonne **subscriptionName**.

## <a name="pay-your-bill"></a>Régler votre facture

En bas de la facture, vous trouverez des instructions pour le règlement de votre facture. Vous pouvez payer par chèque ou par virement dans un délai de 60 jours à compter de la date de votre facture.

Si vous avez déjà payé votre facture, vous pouvez vérifier le statut du paiement dans la page Factures dans le portail Microsoft Azure.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Passer en revue les transactions facturées dans le Portail Azure
> * Passez en revue les frais en attente pour estimer votre prochaine facture
> * Analyser vos frais d’utilisation Azure

Découvrez plus en détail l’utilisation d’Azure Cost Management pour les partenaires.

> [!div class="nextstepaction"]
> [Prise en main d’Azure Cost Management pour les partenaires](../costs/get-started-partners.md)
