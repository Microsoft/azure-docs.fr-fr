---
title: Bien démarrer avec un compte de facturation dans le cadre d’un Contrat client Microsoft - Azure
description: Apprenez-en davantage sur votre compte de facturation de Contrat client Microsoft, notamment les profils de facturation et les modes de paiement des factures.
author: bandersmsft
ms.reviewer: amberbhargava
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 632fac953a54e661164152cda79082aed2c0bc06
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690474"
---
# <a name="get-started-with-your-microsoft-customer-agreement-billing-account"></a>Prise en main de votre compte de facturation dans le cadre d’un Contrat client Microsoft

Un compte de facturation est créé lorsque vous vous inscrivez pour utiliser Azure. Votre compte de facturation vous permet de gérer vos factures et paiements, et d’effectuer un suivi des coûts. Vous pouvez accéder à plusieurs comptes de facturation. Par exemple, vous pouvez vous être inscrit à Azure pour vos projets personnels. Vous pouvez aussi accéder à Azure via le Contrat Entreprise de votre organisation ou le Contrat client Microsoft. Pour chacun de ces scénarios, vous disposez d’un compte de facturation distinct.

Cet article s'applique à un compte de facturation associé à un contrat client Microsoft. [Vérifiez que vous avez accès à un contrat client Microsoft](#check-access-to-a-microsoft-customer-agreement).

## <a name="your-billing-account"></a>Votre compte de facturation

Votre compte de facturation pour le Contrat client Microsoft contient un ou plusieurs profils de facturation qui vous permettent de gérer vos factures et modes de paiement. Chaque profil de facturation contient une ou plusieurs sections de facture qui vous permettent d'organiser les coûts sur la facture du profil de facturation.

Le diagramme suivant montre la relation entre le compte de facturation, les profils de facturation et les sections de la facture.

![Diagramme illustrant la hiérarchie de facturation pour un Contrat client Microsoft](./media/mca-overview/mca-billing-hierarchy.png)

Les rôles du compte de facturation disposent du plus haut niveau d’autorisations. Par défaut, seul l’utilisateur inscrit auprès de Microsoft Azure peut accéder au compte de facturation. Ces rôles sont à attribuer aux utilisateurs qui doivent consulter des factures et suivre les coûts pour toute votre organisation, comme les responsables des services financiers et informatiques. Pour plus d’informations, consultez [Rôles et tâches liés au compte de facturation](../manage/understand-mca-roles.md#billing-account-roles-and-tasks).

## <a name="billing-profiles"></a>Profils de facturation

Utilisez un profil de facturation pour gérer vos factures et modes de paiement. Une facture mensuelle est générée au début du mois pour chaque profil de facturation associé à votre compte. La facture contient les frais respectifs pour tous les abonnements Azure et les autres achats du mois précédent.

Un profil de facturation est automatiquement créé pour votre compte de facturation. Il contient une section de facture par défaut. Vous pouvez créer des sections supplémentaires pour effectuer un suivi et allouer facilement les coûts en fonction de vos besoins, que ce soit par projet, service ou environnement de développement. Ces sections figurent sur la facture et reflètent l’utilisation de chaque abonnement et des achats que vous lui avez attribués.

Les rôles associés aux profils de facturation sont autorisés à afficher et à gérer les factures et les modes de paiement. Attribuez ces rôles aux utilisateurs qui paient les factures tels que les membres du service de comptabilité de votre organisation. Pour plus d’informations, consultez [Rôles et tâches liés au profil de facturation](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

### <a name="each-billing-profile-gets-a-monthly-invoice"></a>Chaque profil de facturation reçoit une facture mensuelle

Une facture mensuelle est générée au début du mois pour chaque profil de facturation. Elle inclut tous les frais du mois précédent.

Dans le portail Azure, vous pouvez afficher la facture, télécharger des documents et modifier le paramètre pour obtenir les futures factures par e-mail. Pour plus d'informations, veuillez consulter [Télécharger les factures pour un contrat client Microsoft](../manage/download-azure-invoice-daily-usage-date.md#download-invoices-for-a-microsoft-customer-agreement).

### <a name="invoice-payment-methods"></a>Modes de paiement des factures

Chaque profil de facturation a ses propres modes de paiement qui permettent de régler les factures associées. Les modes de paiement suivants sont prises en charge :

| Type             | Définition  |
|------------------|-------------|
|Crédits Azure    |  Les crédits sont automatiquement appliqués au montant total facturé de votre facture, ce qui réduit le montant que vous devez payer. Pour plus d’informations, consultez [Suivre le solde du crédit Azure de votre profil de facturation](../manage/mca-check-azure-credits-balance.md). |
|Paiement par virement bancaire/chèque | Si ces types de paiement sont approuvés pour votre compte, vous pouvez payer le montant de votre facture par chèque ou virement. Les instructions de paiement sont indiquées sur la facture |
|Carte de crédit | Les clients qui s’inscrivent auprès de Microsoft Azure via le site web Azure peuvent payer par carte de crédit. |

### <a name="apply-policies-to-control-purchases"></a>Appliquer des stratégies pour contrôler les achats

Vous pouvez appliquer des stratégies afin de contrôler les achats de Réservations et de produits de la Place de marché Microsoft Azure à l’aide d’un profil de facturation. Vous pouvez définir des stratégies pour désactiver l’achat de Réservations Azure et produits de la Place de marché. Lorsque ces stratégies sont appliquées, les abonnements qui sont facturés à un profil de facturation spécifique ne peuvent pas être utilisés pour effectuer ces achats.

### <a name="azure-plans-determine-pricing-and-service-level-agreement-for-subscriptions"></a>Les plans Azure déterminent les contrats de niveau de service et la tarification des abonnements

Les plans Azure déterminent les contrats de niveau de service et la tarification associés aux abonnements. Ils sont automatiquement activés lorsque vous créez un profil de facturation. Toutes les sections de facture associées au profil de facturation peuvent utiliser ces plans. Les utilisateurs ayant accès à la section de facture utilisent les plans pour créer des abonnements Azure. Les plans Azure suivants sont pris en charge dans les comptes de facturation du Contrat client Microsoft :

| Plan             | Définition  |
|------------------|-------------|
|Offre Microsoft Azure   | Autorisez les utilisateurs à créer des abonnements pouvant exécuter n'importe quelle charge de travail.  |
|Offre Microsoft Azure pour Dev/Test | Autorisez les abonnés Visual Studio à créer des abonnements limités aux charges de travail de développement ou de test. Ces abonnements s'accompagnent d'avantages tels que des tarifs privilégiés et un accès à des images de machine virtuelle exclusives dans le portail Azure. |

## <a name="invoice-sections"></a>Sections de facture

Vous pouvez créer des sections de facture pour organiser les coûts affichés sur votre facture. Par exemple, une seule facture peut suffire pour votre organisation, mais vous pouvez souhaitez organiser les coûts par service, équipe ou projet. Pour ce scénario, vous disposez d'un seul profil de facturation dans lequel vous créez une section de facture pour chaque service, équipe ou projet.

Lorsqu’une section de facture est créée, vous pouvez autoriser d’autres utilisateurs à créer des abonnements Azure facturés en fonction de la section. Les frais d’utilisation et les achats liés aux abonnements sont ensuite facturés en fonction de la section.

Les rôles figurant dans la section de facture sont autorisés à contrôler qui crée des abonnements Azure. Attribuez ces rôles aux utilisateurs configurant l’environnement Azure pour les équipes de votre organisation, tels que les responsables d'ingénierie et les architectes techniques. Pour plus d’informations, consultez [Rôles et tâches liés aux sections de facture](../manage/understand-mca-roles.md#invoice-section-roles-and-tasks).

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Vérifier l'accès à un contrat client Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contactez le support technique.

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur votre compte de facturation :

- [Présentation des rôles d'administrateur Azure dans le cadre des Contrats client Microsoft](../manage/understand-mca-roles.md)
- [Créer un abonnement Azure associé dans le cadre de votre Contrat client Microsoft](../manage/create-subscription.md)
- [Créer des sections sur votre facture pour organiser vos coûts](../manage/mca-section-invoice.md)
