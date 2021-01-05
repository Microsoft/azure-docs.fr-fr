---
title: Consulter votre facture Contrat client Microsoft - Azure
description: Découvrez comment consulter votre facture et l’utilisation de vos ressources, et vérifier les frais associés à votre facture Contrat client Microsoft.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: tutorial
ms.date: 12/02/2020
ms.author: banders
ms.openlocfilehash: b5de81f5eaf76c16c1ea8fc78075c4567c77aeee
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531975"
---
# <a name="tutorial-review-your-microsoft-customer-agreement-invoice"></a>Tutoriel : Consulter votre facture Contrat client Microsoft

Vous pouvez consulter les frais sur votre facture en analysant les transactions individuelles. Dans le compte de facturation d’un contrat client Microsoft, une facture est générée chaque mois pour chaque profil de facturation. La facture inclut tous les frais du mois précédent. Vous pouvez voir vos factures dans le portail Azure et comparer les frais au fichier de détails d’utilisation.

Ce tutoriel s’applique uniquement aux clients Azure disposant d’un Contrat client Microsoft.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Passer en revue les transactions facturées dans le Portail Azure
> * Passez en revue les frais en attente pour estimer votre prochaine facture
> * Analyser vos frais d’utilisation Azure

## <a name="prerequisites"></a>Prérequis

Vous devez avoir un compte de facturation associé à un Contrat client Microsoft.

Vous devez avoir accès à un Contrat client Microsoft. Vous devez être Gestionnaire de factures, Propriétaire, Contributeur ou Lecteur de profil de facturation pour voir les informations de facturation et d’utilisation. Pour en savoir plus sur les rôles de facturation pour les contrats client Microsoft, consultez [Tâches et rôles du profil de facturation](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

Cela doit faire plus de 30 jours que vous êtes inscrit à Azure. Azure vous facture à la fin de votre période de facturation.

## <a name="sign-in-to-azure"></a>Connexion à Azure

- Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Vérifier l'accès à un contrat client Microsoft

Vérifiez le type de contrat pour déterminer si vous avez accès au compte de facturation d’un contrat client Microsoft.

Dans le portail Azure, tapez *gestion des coûts + facturation* dans la zone de recherche, puis sélectionnez **Gestion des coûts + facturation**.

![Capture d’écran montrant la recherche de « gestion des coûts + facturation » dans le Portail Azure.](./media/review-customer-agreement-bill/billing-search-cost-management-billing.png)

Si vous avez accès à une seule étendue de facturation, sélectionnez **Propriétés** dans la partie gauche. Vous avez accès au compte de facturation d’un contrat client Microsoft si le type de compte de facturation est **Contrat client Microsoft**.

![Capture d’écran montrant la page de propriétés d’un Contrat client Microsoft](./media/review-customer-agreement-bill/billing-mca-property.png)

Si vous avez accès à plusieurs étendues de facturation, vérifiez le type indiqué dans la colonne du compte de facturation. Vous avez accès à un compte de facturation d’un contrat client Microsoft si le type de compte de facturation de l’une des étendues est **Contrat client Microsoft**.

Dans la page des étendues de facturation, sélectionnez **Étendue de facturation**, puis sélectionnez le compte de facturation, qui sera utilisé pour payer l’utilisation des abonnements. Le compte de facturation doit être de type **Contrat client Microsoft**.

![Capture d’écran montrant un Contrat client Microsoft dans la page listant les comptes de facturation](./media/review-customer-agreement-bill/billing-mca-in-the-list.png)

## <a name="review-invoiced-transactions-in-the-azure-portal"></a>Passer en revue les transactions facturées dans le portail Azure.

Dans le portail Azure, sélectionnez **Toutes les transactions** sur le côté gauche de la page. Selon votre accès, vous devrez peut-être sélectionner un compte, un profil ou une section de facturation, puis **Toutes les transactions**.

La page Toutes les transactions affiche les informations suivantes :

![Capture d’écran montrant la liste des transactions facturées](./media/review-customer-agreement-bill/mca-billed-transactions-list.png)

|Colonne  |Définition  |
|---------|---------|
|Date     | Date de transaction  |
|ID de facture     | L’identificateur de la facture où la transaction a été facturée. Si vous envoyez une demande de support, partagez l’ID avec le support Azure pour accélérer le traitement de votre demande |
|Type de transaction     |  Il existe différents types de transactions : des frais d’achats, d’annulation et d’utilisation  |
|Famille de produits     | La catégorie du produit, tel que le calcul pour les machines virtuelles ou la base de données pour Azure SQL Database|
|Référence SKU de produit     | Un code unique identifiant l’instance de votre produit |
|Montant     |  Le montant de la transaction      |
|Section de facture     | La transaction s’affiche sur cette section de la facture du profil de facturation |
|Profil de facturation     | La transaction s’affiche sur cette facture du profil de facturation |

Recherchez un ID de facture pour filtrer les transactions de la facture.

### <a name="view-transactions-by-invoice-sections"></a>Afficher les transactions en sections de facture

Les sections de facture vous permettent d’organiser les coûts pour la facture d’un profil de facturation. Pour plus d’informations, consultez Lorsqu’une facture est générée, les frais pour toutes les sections du profil de facturation sont répercutés sur la facture.

L’image suivante montre les frais de la section Facture du service de comptabilité sur un exemple de facture.

![Image d’un exemple présentant les informations détaillées d’une facture organisées en sections](./media/review-customer-agreement-bill/invoicesection-details.png)

Lorsque vous avez identifié les frais dans la section d’une facture, vous pouvez afficher les transactions dans le portail Microsoft Azure pour comprendre ces frais.

Dans le portail Microsoft Azure, accédez à la page Toutes les transactions, pour afficher les transactions d’une facture.

Filtrez par nom de section de la facture pour consulter les transactions.

## <a name="review-pending-charges-to-estimate-your-next-invoice"></a>Passez en revue les frais en attente pour estimer votre prochaine facture

Dans le compte de facturation d’un Contrat client Microsoft, tant que les frais ne sont pas facturés, ils sont considérés comme une estimation en attente de validation. Vous pouvez consulter les frais en attente dans le portail Microsoft Azure pour estimer le montant de votre prochaine facture. Les frais en attente sont des estimations et s’entendent hors taxes. Les frais réels figurant sur votre prochaine facture seront différents des frais en attente.

### <a name="view-summary-of-pending-charges"></a>Afficher un résumé des frais en attente

Connectez-vous au [portail Azure](https://portal.azure.com).

Sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation. Dans le compte de facturation, sélectionnez **Profils de facturation**, puis un profil de facturation.

Sélectionnez l’onglet **Résumé** en haut de l’écran.

La section Frais affiche le mois en cours et les frais du mois dernier.

![Capture d’écran illustrant le résumé d’un profil de facturation.](./media/review-customer-agreement-bill/mca-billing-profile-summary.png)

Les frais du mois en cours sont des frais en attente. Ils seront facturés lors de la génération de la facture à la fin du mois. Si la facture du mois dernier n’est pas encore générée, les frais du mois dernier sont également en attente et seront reportés sur votre prochaine facture.

### <a name="view-pending-transactions"></a>Afficher les transactions en attente

Lorsque vous avez identifié les frais en attente, vous pouvez les comprendre en analysant les transactions individuelles qui forment ces frais. À ce stade, les frais d’utilisation en attente ne sont pas affichés sur la page Toutes les transactions. Vous pouvez les afficher sur la page Abonnements Azure.

Connectez-vous au [portail Azure](https://portal.azure.com).

Dans le portail Azure, tapez *gestion des coûts + facturation* dans la zone de recherche, puis sélectionnez **Gestion des coûts + facturation**.

Sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation. Dans le compte de facturation, sélectionnez **Profils de facturation**, puis un profil de facturation.

Sur le côté gauche de la page, sélectionnez **Toutes les transactions**.

Recherchez *En attente*. Utilisez le filtre **Timespan** (Période) pour afficher les frais en attente pour le mois actuel ou le mois dernier.

![Capture d’écran montrant la liste des transactions en attente](./media/review-customer-agreement-bill/mca-pending-transactions-list.png)

### <a name="view-pending-usage-charges"></a>Afficher les frais d’utilisation en attente

Connectez-vous au [portail Azure](https://portal.azure.com).

Dans le portail Azure, tapez *gestion des coûts + facturation* dans la zone de recherche, puis sélectionnez **Gestion des coûts + facturation**.

Sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner un compte de facturation. Dans le compte de facturation, sélectionnez **Profils de facturation**, puis un profil de facturation.

Sur le côté gauche de la page, sélectionnez **Tous les abonnements**.

La page Abonnements Azure affiche les frais actuels et ceux du mois dernier pour chaque abonnement dans le profil de facturation. Les frais du mois en cours sont des frais en attente. Ils seront facturés lors de la génération de la facture à la fin du mois. Si la facture du mois dernier n’est pas encore générée, les frais du mois dernier sont également en attente.

![Capture d’écran montrant les abonnements avec les frais en cumul mensuel à ce jour et les frais du mois précédent.](./media/review-customer-agreement-bill/mca-billing-profile-subscriptions-list.png)

## <a name="analyze-your-azure-usage-charges"></a>Analyser vos frais d’utilisation Azure

Utilisez le fichier CSV Utilisation et frais Azure pour analyser vos frais en fonction de votre utilisation. Vous pouvez télécharger le fichier pour une facture ou des frais en attente.

### <a name="download-your-invoice-and-usage-details"></a>Télécharger les détails de facturation et d’utilisation

En fonction de votre type d’accès, vous devrez peut-être sélectionner un compte de facturation ou un profil de facturation dans Gestion des coûts + facturation. Dans le menu de gauche, sélectionnez **Factures** sous **Facturation**. Dans la grille des factures, recherchez la ligne de la facture que vous souhaitez télécharger. Sélectionnez le symbole de téléchargement ou les points de suspension (...) à la fin de la ligne. Dans la zone **Télécharger**, téléchargez le fichier de détails d’utilisation et la facture.

### <a name="view-detailed-usage-by-invoice-section"></a>Afficher l’utilisation détaillée par section de la facture

Vous pouvez filtrer le fichier Utilisation et frais Azure pour rapprocher les frais d’utilisation des sections de votre facture.

Les information suivantes vous guident lors du rapprochement des frais de calcul pour la section Facture du service de comptabilité :

![Image d’un exemple présentant les informations détaillées d’une facture organisées en sections](./media/review-customer-agreement-bill/invoicesection-details.png)

| Facture PDF | Fichier CSV Utilisation et frais Azure |
| --- | --- |
|Service de comptabilité |invoiceSectionName |
|Frais d’utilisation : Offre Microsoft Azure |productOrderName |
|Calcul |serviceFamily |

Filtre la colonne **invoiceSectionName** dans le fichier CSV sur **Service de comptabilité**. Ensuite, filtrez la colonne **productOrderName** dans le fichier CSV sur **Microsoft Azure Plan**. Ensuite, filtrez la colonne **serviceFamily** dans le fichier CSV sur **Microsoft.Compute**.

![Capture d’écran qui illustre le fichier d’utilisation et de frais filtré par section de facture](./media/review-customer-agreement-bill/billing-usage-file-filtered-by-invoice-section.png)

### <a name="view-detailed-usage-by-subscription"></a>Afficher l’utilisation détaillée par abonnement

Vous pouvez filtrer le fichier CSV Utilisation et frais Azure pour rapprocher les frais d’utilisation de vos abonnements. Lorsque vous avez identifié des frais d’un abonnement, utilisez le fichier CSV Utilisation et frais Azure pour analyser les frais.

L’image suivante montre la liste des abonnements dans le portail Microsoft Azure.

![Capture d’écran montrant la liste des abonnements dans le Portail Azure avec un abonnement encadré.](./media/review-customer-agreement-bill/mca-billing-profile-subscriptions-list-highlighted.png)

Filtre la colonne **subscriptionName** dans le fichier CSV Utilisation et frais Azure pour **WA_Subscription** afin d’afficher les frais d’utilisation détaillés pour WA_Subscription.

![Capture d’écran qui illustre le fichier Utilisation et frais, filtré par abonnement](./media/review-customer-agreement-bill/billing-usage-file-filtered-by-subscription.png)

## <a name="pay-your-bill"></a>Régler votre facture

En bas de la facture, vous trouverez des instructions pour le règlement de votre facture. [Découvrez comment payer](mca-understand-your-invoice.md#how-to-pay).

Si vous avez déjà payé votre facture, vous pouvez vérifier le statut du paiement dans la page Factures dans le portail Microsoft Azure.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Passer en revue les transactions facturées dans le Portail Azure
> * Passez en revue les frais en attente pour estimer votre prochaine facture
> * Analyser vos frais d’utilisation Azure

Suivez le guide de démarrage rapide pour commencer à utiliser l’analyse des coûts.

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts](../costs/quick-acm-cost-analysis.md)
