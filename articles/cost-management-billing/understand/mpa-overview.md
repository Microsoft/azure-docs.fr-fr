---
title: Bien démarrer avec votre compte de facturation dans le cadre d’un Contrat Partenaire Microsoft - Azure CSP
description: Comprendre le fonctionnement de votre compte de facturation associé à un Contrat Partenaire Microsoft (CSP)
author: bandersmsft
ms.reviewer: amberb
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 4dff56db9f1450ff7eb2a2b9ca6f4ca648e2ac38
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150084"
---
# <a name="get-started-with-your-microsoft-partner-agreement-billing-account"></a>Bien démarrer avec votre compte de facturation dans le cadre d’un Contrat Partenaire Microsoft

Un compte de facturation est créé lorsque vous vous inscrivez pour utiliser Azure. Votre compte de facturation vous permet de gérer vos factures et paiements, et d’effectuer un suivi des coûts. Vous pouvez accéder à plusieurs comptes de facturation. Par exemple, vous pouvez vous être inscrit à Azure pour vos projets personnels. Vous pouvez aussi accéder à Azure dans le cadre d’un Contrat Entreprise de votre organisation, d’un Contrat Client Microsoft ou d’un Contrat Partenaire Microsoft. Pour chacun de ces scénarios, vous disposez d’un compte de facturation distinct.

Cet article concerne les comptes de facturation associés aux Contrats Partenaire Microsoft. Ces comptes sont créés pour permettre aux fournisseurs de solutions Cloud (ou CSP) de gérer la facturation de leurs clients dans la nouvelle expérience de commerce. La nouvelle expérience est uniquement disponible pour les partenaires ayant au moins un client qui a accepté un Contrat client Microsoft et qui possède un plan Azure. [Vérifiez si vous avez accès à un Contrat Partenaire Microsoft](#check-access-to-a-microsoft-partner-agreement). Le [plan Azure](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) permet aux clients d'accéder aux services Azure aux tarifs Paiement à l'utilisation en vertu d'un Contrat client Microsoft.

## <a name="your-billing-account"></a>Votre compte de facturation

Votre compte de facturation associé au Contrat Partenaire Microsoft contient un profil de facturation pour chaque devise dans laquelle vous effectuez des transactions commerciales. Le profil de facturation vous permet de gérer vos factures dans la monnaie locale correspondante. Quand vous établissez des relations commerciales avec des clients, en fonction de leurs monnaies locales, les abonnements Azure et les autres achats sont facturés sur les profils de facturation respectifs.

Le diagramme suivant montre la relation entre un compte de facturation, les profils de facturation, les clients et les revendeurs.

![Diagramme illustrant la hiérarchie de facturation pour un Contrat Partenaire Microsoft](./media/mpa-overview/mpa-hierarchy.svg)

Les utilisateurs qui ont le rôle **Administrateur général** ou **Agent d’administration** dans votre organisation ont la possibilité de gérer les comptes de facturation, les profils de facturation et les clients. Pour plus d’informations, consultez [Espace partenaires - Affecter des rôles et des autorisations aux utilisateurs](/partner-center/permissions-overview).

## <a name="billing-profiles"></a>Profils de facturation

Utilisez un profil de facturation pour gérer vos factures dans une devise. Une facture mensuelle est générée au début du mois pour chaque profil de facturation associé à votre compte. La facture inclut les frais dans la devise du profil de facturation pour tous les abonnements Azure et les autres achats du mois précédent.

Vous pouvez afficher la facture et télécharger les documents associés, comme le fichier d’utilisation et la grille tarifaire, dans le portail Azure. Pour plus d’informations, consultez [Télécharger les factures pour un Contrat Partenaire Microsoft](download-azure-invoice.md).

> [!IMPORTANT]
>
> Les factures pour les profils de facturation incluent les frais facturés aux clients qui ont des plans Azure, ainsi que tous les achats (de réservation, sur SaaS et sur la Place de marché Azure) effectués par les clients qui n’ont pas accepté le Contrat Client Microsoft et qui n’ont pas de plans Azure.

## <a name="customers"></a>Clients

Vous pouvez voir et gérer les clients qui ont accepté un Contrat Client Microsoft et qui ont un plan Azure dans le portail Azure. Vous pouvez aussi voir les frais et les transactions, ainsi que créer et gérer les abonnements Azure pour ces clients.

### <a name="enable-policy-to-give-visibility-into-cost"></a>Activer la stratégie pour avoir une meilleure visibilité du coût

Appliquez la stratégie pour contrôler si les utilisateurs dans l’organisation des clients peuvent voir et analyser le coût aux tarifs du paiement à l’utilisation pour leur consommation d’Azure. Par défaut, la stratégie est désactivée et les utilisateurs ne peuvent pas voir le coût. Quand cette stratégie est activée, les utilisateurs qui disposent d’un accès [Azure RBAC](../../role-based-access-control/overview.md) approprié sur un abonnement peuvent voir et analyser le coût de l’abonnement.

Pour activer la stratégie :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Effectuez une recherche sur **Gestion des coûts + facturation**.

   ![Capture d’écran montrant la recherche de « gestion des coûts + facturation » dans le Portail Azure.](./media/mpa-overview/search-cmb.png)

1. Sélectionnez **Clients** dans la partie gauche, puis sélectionnez un client dans la liste.

   ![Capture d’écran montrant la sélection d’un client](./media/mpa-overview/mpa-customers.png)

1. Sélectionnez **Stratégies** dans la partie gauche.

   ![Capture d’écran montrant les stratégies](./media/mpa-overview/mpa-change-policy.png)

1. Sélectionnez **Oui**.

## <a name="resellers"></a>Revendeurs

Les fournisseurs indirects dans le [modèle CSP de niveau deux](/partner-center) peuvent sélectionner un revendeur quand ils créent des abonnements pour les clients dans le portail Azure. Après la création, ils peuvent voir la liste des abonnements filtrés par un revendeur et analyser le coût pour un client par les revendeurs dans l’analyse des coûts Azure.

## <a name="check-access-to-a-microsoft-partner-agreement"></a>Vérifier l’accès à un Contrat Partenaire Microsoft
[!INCLUDE [billing-check-mpa](../../../includes/billing-check-mpa.md)]

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur votre compte de facturation :

- [Créer un abonnement Azure supplémentaire dans le cadre de votre Contrat Partenaire Microsoft](../manage/create-subscription.md)
- Intégrer les données de facturation à votre propre système de rapports à l’aide des [API Facturation Azure](/rest/api/billing/)
- [Guide de démarrage rapide d’Azure Cost Management pour les partenaires](../costs/get-started-partners.md)