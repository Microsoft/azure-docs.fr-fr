---
title: Tâches EA dans un Contrat client Microsoft - Azure
description: Découvrez comment effectuer des tâches Contrat Entreprise dans votre nouveau compte de facturation.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 09/03/2020
ms.author: banders
ms.openlocfilehash: e5b1ecfd7b434ceefcff65178b7532575a759d45
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440624"
---
# <a name="complete-enterprise-agreement-tasks-in-your-billing-account-for-a-microsoft-customer-agreement"></a>Effectuer des tâches Contrat Entreprise dans votre compte de facturation pour un contrat client Microsoft

Si votre organisation a signé un Contrat client Microsoft pour renouveler votre accord de mise en œuvre EA, un nouveau compte de facturation est créé pour cet accord. La facturation de votre nouveau compte est organisée différemment de votre Contrat Entreprise. Cet article décrit comment vous pouvez utiliser le nouveau compte de facturation pour effectuer les tâches que vous effectuiez dans votre Contrat Entreprise.

## <a name="billing-organization-in-the-new-account"></a>Organisation de la facturation dans le nouveau compte

Le schéma suivant explique comment la facturation est organisée dans le nouveau compte de facturation.

![Image de la hiérarchie-post-transition-mca-ea](./media/mca-enterprise-operations/mca-post-transition-hierarchy.png)

| Contrat Entreprise   | Contrat client Microsoft    |
|------------------------|--------------------------------------------------------|
| Inscription            | Le profil de facturation vous permet de gérer la facturation de votre organisation, de la même façon que votre accord de mise en œuvre EA. Les administrateurs d’entreprise deviennent propriétaires du profil de facturation. Pour en savoir plus sur les profils de facturation, consultez [Comprendre les profils de facturation](../understand/mca-overview.md#billing-profiles).
| department            | Les sections de facture permettent d’organiser vos coûts, de la même façon que les départements dans votre accord de mise en œuvre EA. Les départements deviennent les sections de facture et les administrateurs de départements deviennent les propriétaires des sections de facture respectives. Pour en savoir plus sur les sections de la facture, consultez [Comprendre les sections de la facture](../understand/mca-overview.md#invoice-sections). |
| Compte               | Les comptes qui ont été créés dans votre Contrat Entreprise ne sont pas pris en charge dans le nouveau compte de facturation. Les abonnements du compte appartiennent à la section de facture respective du département. Les propriétaires de comptes peuvent créer et gérer des abonnements pour leurs sections de facture. |

## <a name="changes-for-enterprise-administrators"></a>Modifications apportées aux administrateurs d’entreprise

Les modifications suivantes s’appliquent aux administrateurs d’entreprise sur un Contrat Entreprise qui a été renouvelé pour un Contrat client Microsoft.

- Un profil de facturation est créé pour votre accord de mise en œuvre. Le profil de facturation vous permet de gérer la facturation de votre organisation, de la même façon que votre inscription à un Contrat Entreprise. Pour en savoir plus sur les profils de facturation, consultez [Comprendre les profils de facturation](../understand/mca-overview.md#billing-profiles).
- Une section de la facture est créée pour chaque département dans votre accord de mise en œuvre EA. Les sections de facture vous permettent de gérer vos services. Vous pouvez créer de nouvelles sections de facture pour configurer d’autres départements. Pour en savoir plus sur les sections de facture, consultez [Comprendre les sections de facture](../understand/mca-overview.md#invoice-sections).
- Vous utiliserez le rôle de créateur d’abonnement Azure sur les sections de facture pour autoriser d’autres utilisateurs à créer des abonnements Azure, par exemple les comptes qui ont été créés dans l’inscription au Contrat Entreprise.
- Vous pouvez utiliser le [portail Azure](https://portal.azure.com) plutôt que le portail Azure EA pour gérer la facturation de votre organisation.

Vous disposez des rôles suivants sur le nouveau compte de facturation :

**Propriétaire de profil de facturation** : le rôle de propriétaire du profil de facturation du profil de facturation qui a été créé lors de la signature de l’accord vous est affecté. Ce rôle vous permet de gérer la facturation de votre organisation. Vous pouvez consulter les frais et les factures, organiser les coûts sur la facture, gérer les modes de paiement et contrôler l’accès à la facturation de votre organisation.

**Propriétaire de section de facture** : le rôle de propriétaire de la section de facture vous est affecté sur toutes les sections de facture créées pour les départements de votre accord de mise en œuvre EA. Le rôle vous permet de contrôler qui peut créer des abonnements Azure et acheter d’autres produits.

### <a name="view-charges-and-credits-balance-for-your-organization"></a>Consulter les frais et le solde des crédits de votre organisation

Le profil de facturation vous permet de suivre les frais et le solde de crédits Azure de votre organisation, de la même façon que votre accord de mise en œuvre EA.

Pour savoir comment consulter le solde de crédit pour un profil de facturation, consultez [Suivre le solde du crédit Azure de votre profil de facturation](mca-check-azure-credits-balance.md).

Pour savoir comment consulter les frais pour un profil de facturation, consultez [Comprendre les frais sur votre facture de Contrat client Microsoft](../understand/review-customer-agreement-bill.md).

### <a name="view-charges-for-a-department"></a>Consulter les frais pour un département

Une section de facture est créée pour chaque département que vous aviez dans votre Contrat Entreprise. Vous pouvez voir les frais d’une section de facture dans le portail Azure. Pour plus d’informations, consultez [Voir les transactions par section de facture](../understand/review-customer-agreement-bill.md#view-transactions-by-invoice-sections).

### <a name="view-charges-for-an-account"></a>Voir les frais d’un compte

Les comptes qui ont été créés dans votre accord de mise en œuvre EA ne sont pas pris en charge dans le nouveau compte de facturation. Les abonnements du compte appartiennent à la section de facture respective du département. Les propriétaires de comptes peuvent créer et gérer des abonnements pour leurs sections de facture.

Pour voir le coût cumulé pour les abonnements appartenant à un compte, vous devez définir un centre de coûts pour chaque abonnement. Ensuite, vous pouvez utiliser le fichier CSV d’utilisation et de frais Azure pour filtrer les abonnements par centre de coûts.

### <a name="download-usage-and-charges-csv-price-sheet-and-tax-documents"></a>Télécharger le fichier CSV d’utilisation et de frais, la grille tarifaire et les documents fiscaux

Une facture mensuelle est générée pour chaque profil de facturation dans votre compte de facturation. Pour chaque facture, vous pouvez télécharger le fichier CSV d’utilisation et de frais Azure, la grille tarifaire et le document fiscal (le cas échéant). Vous pouvez également télécharger les fichiers CSV d’utilisation et de frais Azure pour les frais du mois en cours.

Pour savoir comment télécharger les fichiers CSV d’utilisation et de frais Azure, consultez [Télécharger les informations d’utilisation pour votre Contrat client Microsoft](../understand/download-azure-daily-usage.md).

Pour savoir comment télécharger la grille tarifaire, consultez [Télécharger la tarification pour votre Contrat client Microsoft](ea-pricing.md).

Pour savoir comment télécharger les documents fiscaux, consultez [Afficher les documents fiscaux pour votre Contrat client Microsoft](../understand/mca-download-tax-document.md#view-and-download-tax-documents).

### <a name="add-an-additional-enterprise-administrator"></a>Ajouter un autre administrateur d’entreprise

Donnez aux utilisateurs l’accès au profil de facturation pour leur permettre de consulter et de gérer la facturation de votre organisation. Vous pouvez utiliser la page **Contrôle d’accès (IAM)** sur le portail Azure pour accorder les accès.  Pour en savoir plus sur les rôles de profil de facturation, consultez [Rôles et tâches liés au profil de facturation](understand-mca-roles.md#billing-profile-roles-and-tasks).

Pour savoir comment fournit un accès à votre profil de facturation, consultez [Gérer les rôles de facturation sur le portail Azure](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="create-a-new-department"></a>Créer un département

Créez une section de facture pour organiser vos coûts en fonction de vos besoins, de la même façon que les départements dans votre accord de mise en œuvre EA. Vous pouvez créer une section de facture dans le portail Azure. Pour plus d’informations, consultez [Créer des sections sur votre facture pour organiser vos coûts](mca-section-invoice.md).

### <a name="create-a-new-account"></a>Créer un nouveau compte

Affectez aux utilisateurs le rôle de créateur d’abonnement Azure sur les sections de facture pour les autoriser à créer des abonnements Azure, par exemple les comptes qui ont été créés dans l’accord de mise en œuvre EA. Pour plus d’informations sur l’attribution de rôles, consultez [Gérer les rôles de facturation dans le portail Azure](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal)

## <a name="changes-for-department-administrators"></a>Modifications apportées aux administrateurs de service

Les modifications suivantes s’appliquent aux administrateurs de département sur un Contrat Entreprise qui a été renouvelé pour un Contrat client Microsoft.

- Une section de la facture est créée pour chaque département dans votre accord de mise en œuvre EA. La ou les sections de facture vous permettent de gérer vos services. Pour en savoir plus sur les sections de facture, consultez [Comprendre les sections de facture](../understand/mca-overview.md#invoice-sections).
- Vous utiliserez le rôle de créateur d’abonnement Azure sur la section de facture pour autoriser d’autres utilisateurs à créer des abonnements Azure, par exemple les comptes qui ont été créés dans l’inscription au Contrat Entreprise.
- Vous pouvez utiliser le portail Azure plutôt que le portail Azure EA pour gérer la facturation de votre organisation.

Vous disposez des rôles suivants sur le nouveau compte de facturation :

**Propriétaire de section de facture** : le rôle de propriétaire de la section de facture vous est affecté sur la section de facture créée pour le ou les départements de votre accord de mise en œuvre EA. Ce rôle vous permet de voir et de suivre les frais et de contrôler qui peut créer des abonnements Azure et acheter d’autres produits pour cette section de la facture.

### <a name="view-charges-for-your-department"></a>Consulter les frais pour votre département

Vous pouvez consulter les frais pour la section de facture créée pour votre service, dans la [page Gestion des coûts + facturation](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/Overview) du portail Azure.

### <a name="add-an-additional-department-administrator"></a>Ajouter un autre administrateur de département

Une section de facture est créée pour chaque département que vous aviez dans votre Contrat Entreprise. Vous pouvez utiliser la page **Contrôle d’accès (IAM)** du portail Azure pour que d’autres personnes puissent afficher et gérer la section de facture. Pour en savoir plus sur les rôles de section de facture, consultez [Rôles et tâches liés aux sections de facture](understand-mca-roles.md#invoice-section-roles-and-tasks).

Pour savoir comment fournit un accès à votre section de facture, consultez [Gérer les rôles de facturation sur le portail Azure](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="create-a-new-account-in-your-department"></a>Créer un compte dans votre département

Affectez le rôle de créateur d’abonnement Azure sur la section de facture créée pour votre service. Pour plus d’informations sur l’attribution de rôles, consultez [Gérer les rôles de facturation dans le portail Azure](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal)

### <a name="view-charges-for-accounts-in-your-departments"></a>Consulter les frais pour les comptes dans vos départements

Les comptes qui ont été créés dans votre accord de mise en œuvre EA ne sont pas pris en charge dans le nouveau compte de facturation. Les abonnements du compte appartiennent à la section de facture respective du département. Les propriétaires de comptes peuvent créer et gérer des abonnements pour leurs sections de facture.

Pour voir le coût cumulé pour les abonnements appartenant à un compte de votre département, vous devez définir un centre de coûts pour chaque abonnement. Ensuite, vous pouvez utiliser le fichier d’utilisation et de frais Azure pour filtrer les abonnements par centre de coûts.

## <a name="changes-for-account-owners"></a>Modifications apportées aux propriétaires de compte

Les propriétaires de comptes sur le Contrat Entreprise obtiennent l’autorisation de créer des abonnements Azure sur le nouveau compte de facturation. Vos abonnements Azure existants appartiennent à la section de facture qui est créée pour votre département. Si votre compte n’appartient à aucun service, vos abonnements appartiennent à une section de facture appelée « Default invoice section » (Section de facture par défaut).

Pour créer d’autres abonnements Azure, vous obtenez le rôle suivant sur le nouveau compte de facturation.

**Créateur de l’abonnement Azure** : le rôle de créateur d’abonnement Azure vous est affecté dans la section de facture qui est créée pour votre département dans le Contrat Entreprise. Si votre compte n’appartient à aucun département, vous bénéficiez du rôle de créateur d’abonnement Azure sur une section de facture appelée Section de facture par défaut. Ce rôle vous permet de créer les abonnements Azure associés à la section de facture.

### <a name="create-an-azure-subscription"></a>Crée un abonnement Azure

Vous pouvez créer des abonnements Azure pour votre section de facture dans le portail Azure. Pour plus d’informations, consultez [Créer un abonnement Azure associé dans le cadre de votre Contrat client Microsoft](create-subscription.md).

### <a name="view-charges-for-your-account"></a>Consulter les frais pour votre compte

Pour consulter les frais relatifs aux abonnements appartenant à un compte, accédez à la [page des abonnements](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) dans le portail Azure. La page des abonnements affiche les frais pour tous vos abonnements.

### <a name="view-charges-for-a-subscription"></a>Consulter les frais pour un abonnement

Vous pouvez consulter les frais pour un abonnement soit sur la [page des abonnements](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade), soit dans l’analyse des coûts Azure. Pour plus d’informations sur l’analyse des coûts Azure, consultez [Explorer et analyser les coûts avec l’analyse des coûts](../costs/quick-acm-cost-analysis.md).

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre le compte de facturation associé à un Contrat client Microsoft](../understand/mca-overview.md)
- [Comprendre votre facture](../understand/review-individual-bill.md)
- [Comprendre votre facture](../understand/understand-invoice.md)
- [Obtenir la propriété de facturation des abonnements Azure des autres utilisateurs](mca-request-billing-ownership.md)
