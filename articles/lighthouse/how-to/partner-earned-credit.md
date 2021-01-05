---
title: Lier votre ID de partenaire pour suivre votre impact sur les ressources déléguées
description: Découvrez comment associer votre ID de partenaire pour utiliser le crédit Partenaires sur des ressources de clients que vous gérez par le biais d’Azure Lighthouse.
ms.date: 10/30/2020
ms.topic: how-to
ms.openlocfilehash: fcbcc70e380116b8e9f9b1c1e365dee1adb87a99
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080275"
---
# <a name="link-your-partner-id-to-track-your-impact-on-delegated-resources"></a>Lier votre ID de partenaire pour suivre votre impact sur les ressources déléguées 

Si vous êtes membre du [Microsoft Partner Network](https://partner.microsoft.com/), vous pouvez lier votre ID de partenaire à des informations d’identification utilisées pour gérer des ressources de clients déléguées. Partner Admin Link (PAL) permet à Microsoft d’identifier et de gratifier les partenaires qui favorisent la réussite des clients Azure. Ce lien permet également aux partenaires [CSP (fournisseur de solutions Cloud)](/partner-center/csp-overview) de recevoir un [crédit pour les services gérés (PEC)](/partner-center/partner-earned-credit) pour les clients qui ont [signé Contrat client Microsoft (MCA)](/partner-center/confirm-customer-agreement) et sont [sous le plan Azure](/partner-center/azure-plan-get-started).

Si vous [intégrez des clients à des offres de service gérés dans la Place de marché Azure](publish-managed-services-offers.md), ce lien se produit automatiquement en utilisant l’ID MPN associé au compte Espace partenaires utilisé pour publier les offres. Aucune action supplémentaire n’est nécessaire pour suivre votre impact sur ces clients.

Si vous [intégrez des clients à l’aide de modèles Azure Resource Management](onboard-customer.md), vous devez prendre des mesures pour créer ce lien. Pour cela, [liez votre ID MPN](../../cost-management-billing/manage/link-partner-id.md) avec au moins un compte d’utilisateur dans votre locataire qui a accès à chacun de vos abonnements intégrés.

## <a name="associate-your-partner-id-when-you-onboard-new-customers"></a>Associer votre ID de partenaire lors de l’intégration de nouveaux clients

Lorsque vous intégrez des clients à l’aide de modèles Azure Resource Manager (modèles ARM), utilisez la procédure suivante pour lier votre ID partenaire et activer le crédit Partenaires (si applicable). Vous devez connaître votre [ID de partenaire MPN](/partner-center/partner-center-account-setup#locate-your-mpn-id) pour effectuer ces étapes. Vous devez utiliser l’ **ID MPN associé** qui est indiqué dans votre profil partenaire.

Pour simplifier, nous vous recommandons de créer un compte de principal du service dans votre locataire en l’associant à votre **ID MPN** , puis d’accorder à chaque client que vous intégrez un accès à ce compte avec un [rôle intégré Azure éligible au crédit Partenaires](/partner-center/azure-roles-perms-pec).

1. [Créez un compte de principal de service](../../active-directory/develop/howto-authenticate-service-principal-powershell.md) dans votre locataire gérant. Pour cet exemple, nous allons utiliser le nom *Compte Automation de fournisseur* pour ce principal du service.
1. En utilisant ce compte de principal du service, [établissez un lien avec votre ID MPN](../../cost-management-billing/manage/link-partner-id.md#link-to-a-partner-id) dans votre locataire gérant. Cette opération est unique.
1. Lorsque vous [intégrez un client à l’aide de modèles ARM](onboard-customer.md), veillez à inclure une autorisation qui inclut le Compte Automation de fournisseur en tant qu’utilisateur disposant d’un [rôle intégré Azure éligible au crédit Partenaires](/partner-center/azure-roles-perms-pec).

En suivant ces étapes, chaque locataire de client que vous gérez est associé à votre ID de partenaire. Le Compte Automation de fournisseur n’a pas besoin de s’authentifier ni d’effectuer des actions dans le locataire du client.

:::image type="content" source="../media/lighthouse-pal.jpg" alt-text="Diagramme montrant le processus PAL avec Azure Lighthouse.":::

## <a name="add-your-partner-id-to-previously-onboarded-customers"></a>Ajoute votre ID de partenaire aux clients précédemment intégrés

Si vous avez déjà intégré un client, vous ne voulez peut-être pas effectuer un autre déploiement pour ajouter le principal du service de votre Compte Automation de fournisseur. Vous pouvez plutôt associer votre **ID MPN** à un compte d’utilisateur qui a déjà accès au locataire de ce client. Vérifiez que le compte a reçu un [rôle intégré Azure éligible au crédit Partenaires](/partner-center/azure-roles-perms-pec).

Une fois que le compte a été [lié à votre ID MPN](../../cost-management-billing/manage/link-partner-id.md#link-to-a-partner-id) dans votre locataire gérant, vous pouvez recevoir suivre votre impact sur ce client.

## <a name="confirm-partner-earned-credit"></a>Confirmer le crédit Partenaires

Vous pouvez [voir les détails du crédit Partenaires dans le portail Azure](/partner-center/partner-earned-credit-explanation#azure-cost-management) pour vérifier les coûts qui ont bénéficié de l’avantage de ce crédit Partenaires. N’oubliez pas que le PEC s’applique uniquement aux clients CSP qui ont signé le MCA et qui sont sous le plan Azure.

Si vous avez suivi les étapes ci-dessus et que vous ne voyez pas l’association attendue, ouvrez une demande de support dans le portail Azure.

Vous pouvez également utiliser le [Kit de développement logiciel (SDK) Espace partenaires](/partner-center/develop/get-invoice-unbilled-consumption-lineitems) et filtrer sur `rateOfPartnerEarnedCredit` pour automatiser la vérification du PEC pour un abonnement.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur le [Microsoft Partner Network](/partner-center/mpn-overview).
- Découvrez comment le [crédit Partenaires est calculé et payé](/partner-center/partner-earned-credit-explanation).
- [Intégrez des clients](onboard-customer.md) à Azure Lighthouse.