---
title: Afficher et télécharger les tarifs Azure de votre organisation
description: Découvrez comment afficher et télécharger les tarifs ou une estimation des coûts avec les tarifs de votre organisation.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: 63e648681a63daafe1eaeb1d2e8e5a6cd607aa70
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91446331"
---
# <a name="view-and-download-your-organizations-azure-pricing"></a>Afficher et télécharger les tarifs Azure de votre organisation

Les clients Azure qui ont un Contrat Entreprise Azure (EA), un Contrat Client Microsoft (MCA) ou un Contrat Partenaire Microsoft (MPA) peuvent voir et télécharger leurs tarifs dans le portail Azure. [Découvrez comment vérifier votre type de compte de facturation](#check-your-billing-account-type).

## <a name="download-pricing-for-an-enterprise-agreement"></a>Télécharger les tarifs pour un Contrat Entreprise

Selon les stratégies définies pour votre organisation par l’administrateur d’entreprise, seuls certains rôles d’administrateur permettent d’accéder aux informations tarifaires Contrat Entreprise de votre organisation. Pour plus d’informations, consultez [Comprendre les rôles d’administrateur Contrat Entreprise Azure dans Azure](understand-ea-roles.md).

1. En tant qu’administrateur d’entreprise, connectez-vous au [Portail Azure](https://portal.azure.com/).
1. Recherchez *Gestion des coûts + facturation*.

   ![Capture d’écran montrant une recherche dans le Portail Azure.](./media/ea-pricing/portal-cm-billing-search.png)

1. Sous le compte de facturation, sélectionnez **Utilisation + frais**.

   ![Capture d’écran montrant l’utilisation et les frais sous Facturation](./media/ea-pricing/ea-pricing-usage-charges-nav.png)

1. Sélectionnez l’![icône de téléchargement](./media/ea-pricing/download-icon.png). **Télécharger** pour le mois.

1. Sous **Grille tarifaire**, sélectionnez **Télécharger le fichier CSV**.

   ![Capture d’écran illustrant l’option Télécharger l’utilisation et les frais.](./media/ea-pricing/download-ea-price-sheet.png)

## <a name="download-pricing-for-an-mca-or-mpa-account"></a>Télécharger les tarifs pour un compte MCA ou MPA

Si vous avez un compte MCA, vous devez être responsable de la facturation ou propriétaire, contributeur ou lecteur de profil de facturation pour voir et télécharger les tarifs. Si vous avez un compte MPA, vous devez avoir le rôle Administrateur général ou Agent d’administration dans l’organisation partenaire pour voir et télécharger les tarifs.

### <a name="download-price-sheets-for-billed-charges"></a>Télécharger les grilles tarifaires pour les frais facturés

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez *Gestion des coûts + facturation*.
1. Sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner au préalable un compte de facturation.
1. Sélectionnez **Factures**.
1. Dans la grille des factures, recherchez la ligne de la facture correspondant à la grille tarifaire que vous souhaitez télécharger.
1. Cliquez sur les points de suspension (`...`) à la fin de la ligne.
![Capture d’écran montrant les points de suspension sélectionnés](./media/ea-pricing/billingprofile-invoicegrid-new.png)

1. Si vous souhaitez voir les tarifs des services dans la facture sélectionnée, sélectionnez **Grille tarifaire de facturation**.
1. Si vous souhaitez voir les tarifs de tous les services Azure pour la période de facturation donnée, sélectionnez **Grille tarifaire Azure**.

![Capture d’écran montrant le menu contextuel avec les grilles tarifaires](./media/ea-pricing/contextmenu-pricesheet01.png)

### <a name="download-price-sheets-for-the-current-billing-period"></a>Télécharger les grilles tarifaires pour la période de facturation actuelle

Si vous avez un compte MCA, vous pouvez télécharger les tarifs pour la période de facturation actuelle.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez *Gestion des coûts + facturation*.
1. Sélectionnez un profil de facturation. Selon votre accès, vous devrez peut-être sélectionner au préalable un compte de facturation.
1. Dans la zone **Vue d’ensemble**, recherchez les liens de téléchargement sous les frais en cumul mensuel à ce jour.
1. Sélectionnez **Grille tarifaire Azure**.
![Capture d’écran montrant le téléchargement à partir du panneau Vue d’ensemble](./media/ea-pricing/open-pricing01.png)

## <a name="estimate-costs-with-the-azure-pricing-calculator"></a>Estimer les coûts avec la calculatrice de prix Azure

Vous pouvez également utiliser les tarifs de votre organisation pour estimer les coûts à l’aide de la calculatrice de prix Azure.

1. Accédez à la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator).
1. En haut à droite, sélectionnez **Connexion**.
1. Sous **Programmes et offre** > **Programme de licence**, sélectionnez **Contrat Entreprise**.
1. Sous **Programmes et offre** > **Contrat sélectionné**, sélectionnez **Aucune sélection**.

    ![Capture d’écran montrant les programmes et offres disponibles.](./media/ea-pricing/ea-pricing-calculator-estimate.png)

1. Choisissez l’organisation.
1. Sélectionnez **Appliquer**.
1. Recherchez et ajoutez des produits à votre devis.
1. Les prix estimés affichés sont basés sur les tarifs de l’organisation que vous avez sélectionnés.

## <a name="check-your-billing-account-type"></a>Vérifier votre type de compte de facturation
[!INCLUDE [billing-check-account-type](../../../includes/billing-check-account-type.md)]

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes client EA, consultez :

- [Gérer l’accès aux informations de facturation EA pour Azure](manage-billing-access.md)
- [Afficher et télécharger votre facture Microsoft Azure](../understand/download-azure-invoice.md)
- [Afficher et télécharger votre utilisation et vos frais Microsoft Azure](../understand/download-azure-daily-usage.md)
- [Comprendre votre facture pour les clients Contrat Entreprise](../understand/review-enterprise-agreement-bill.md)

Si vous avez un Contrat client Microsoft, consultez :

- [Comprendre les conditions de votre grille tarifaire pour un Contrat client Microsoft](mca-understand-pricesheet.md)
- [Afficher et télécharger votre facture Microsoft Azure](../understand/download-azure-invoice.md)
- [Afficher et télécharger votre utilisation et vos frais Microsoft Azure](../understand/download-azure-daily-usage.md)
- [Afficher et télécharger les documents fiscaux pour votre profil de facturation](../understand/mca-download-tax-document.md)
- [Comprendre les frais indiqués sur la facture de votre profil de facturation](../understand/review-customer-agreement-bill.md)
