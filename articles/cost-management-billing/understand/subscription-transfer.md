---
title: À propos du transfert de propriété de facturation pour un abonnement Azure
description: Cet article explique ce que vous devez savoir avant de transférer une propriété de facturation d’un abonnement Azure vers un autre compte.
keywords: transférer abonnement azure, azure transfert abonnement, déplacer un abonnement azure vers un autre compte, changer abonnement azure, transférer abonnement azure à un autre compte, transférer facturation azure
author: bandersmsft
ms.reviewer: amberb
tags: billing,top-support-issue
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: 163bd7dbdac4de7b7d9c1657ac3debf45382a27a
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027269"
---
# <a name="about-transferring-billing-ownership-for-an-azure-subscription"></a>À propos du transfert de propriété de facturation pour un abonnement Azure

Cet article vous explique ce que vous devez connaître avant de transférer une propriété de facturation d’un abonnement Azure vers un autre compte. 

Vous pouvez transférer la propriété de la facturation de votre abonnement Azure si vous quittez votre organisation ou si voulez que votre abonnement soit facturé sur un autre compte. Le transfert de la propriété de la facturation à un autre compte fournit aux administrateurs du nouveau compte l’autorisation sur les tâches de facturation. Ils peuvent changer le mode de paiement, visualiser les coûts facturés et annuler l’abonnement.

Si vous souhaitez conserver la propriété de facturation tout en changeant le type de votre abonnement, consultez [Changer d’offre pour votre abonnement Azure](../manage/switch-azure-offer.md). Pour contrôler qui peut accéder aux ressources de l’abonnement, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

Si vous êtes un client disposant d’un Contrat Entreprise (EA), les administrateurs de votre entreprise peuvent transférer une propriété de facturation de vos abonnements entre des comptes.

Seul l’administrateur de facturation d’un compte peut transférer la propriété d’un abonnement.

## <a name="determine-account-billing-administrator"></a>Déterminer l’administrateur de facturation d’un compte

<a name="whoisaa"></a>

L’administrateur de facturation est la personne autorisée à gérer la facturation d’un compte. Il est autorisé à accéder à la facturation sur le [portail Azure](https://portal.azure.com) et à effectuer différentes tâches liées à la facturation, comme la création d’abonnements, la consultation et le paiement des factures, ou la mise à jour des modes de paiement.

Pour identifier les comptes dont vous êtes l’administrateur de facturation, consultez la page [Gestion des coûts + facturation dans le portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/Overview). Dans le volet de gauche, sélectionnez **Toutes les étendues de facturation**. La page des abonnements affiche tous les abonnements dont vous êtes l’administrateur de facturation.

Si vous n’êtes pas certain de l’identité de l’administrateur de compte d’un abonnement, accédez à la [page Abonnements dans le portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). Sélectionnez ensuite l’abonnement que vous souhaitez consulter, puis regardez sous **Paramètres**. Sélectionnez **Propriétés**. L’administrateur de compte de l’abonnement s’affiche dans la zone **Administrateur de compte**.


## <a name="supported-subscription-types"></a>Types d’abonnements pris en charge

Le transfert d’abonnement dans le portail Azure est disponible pour les types d’abonnements listés ci-dessous. Actuellement, le transfert n’est pas pris en charge pour les abonnements [Essai gratuit](https://azure.microsoft.com/offers/ms-azr-0044p/) ou [Azure dans Open (AIO)](https://azure.microsoft.com/offers/ms-azr-0111p/). Pour obtenir une solution de contournement, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md). Pour transférer d’autres abonnements, comme des plans de support, [contactez le support Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

- [Contrat Entreprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)<sup>1</sup>
- [Microsoft Partner Network](https://azure.microsoft.com/offers/ms-azr-0025p/)  
- [Abonnés Visual Studio Enterprise (MPN)](https://azure.microsoft.com/offers/ms-azr-0029p/)
- [Plateformes MSDN](https://azure.microsoft.com/offers/ms-azr-0062p/)  
- [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/)
- [Dev/Test – Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/)
- [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/)
- [Visual Studio Enterprise : BizSpark](https://azure.microsoft.com/offers/ms-azr-0064p/)
- [Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/)
- [Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/)
- [Plan Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0017g/)<sup>2</sup>

<sup>1</sup> Utilisant le portail EA.

<sup>2</sup> Uniquement pris en charge pour les comptes créés lors de l’inscription sur le site web Azure.

## <a name="resources-transferred-with-subscriptions"></a>Ressources transférées avec des abonnements

Toutes les ressources telles que les machines virtuelles, les disques et les sites web sont transférées au nouveau compte. Cependant, si vous transférez un abonnement vers un compte dans un autre locataire Azure AD, les [rôles Administrateur](../manage/add-change-subscription-administrator.md) et les [attributions de rôles Azure](../../role-based-access-control/role-assignments-portal.md) sur l’abonnement ne sont pas transférés. Les [inscriptions des applications](../../active-directory/develop/quickstart-register-app.md) et autres services spécifiques du locataire ne sont pas non plus transférés avec l’abonnement.

## <a name="transfer-account-ownership-to-another-countryregion"></a>Transférer la propriété d’un compte vers un autre pays/région

Malheureusement, vous ne pouvez pas transférer d’abonnements entre pays ou régions à l’aide du portail Azure. Ceux-ci peuvent cependant être transférés si vous ouvrez une demande de support Azure. Pour créer une demande de support, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="transfer-a-subscription-from-one-account-to-another"></a>Transférer un abonnement d’un compte à un autre

Si vous êtes l’administrateur de deux comptes, vous pouvez transférer un abonnement entre vos comptes. Vos comptes étant conceptuellement considérés comme les comptes de deux utilisateurs différents, vous pouvez transférer des abonnements entre eux.
Pour voir la procédure à suivre pour transférer votre abonnement, consultez [Transférer la propriété de facturation d’un abonnement Azure](../manage/billing-subscription-transfer.md).

## <a name="transferring-a-subscription-shouldnt-create-downtime"></a>Le transfert d’un abonnement ne doit pas entraîner de temps d’arrêt

Si vous transférez un abonnement à un compte dans le même locataire Azure AD, il n’y a pas d’impact sur les ressources en cours d’exécution dans l’abonnement. Cependant, les informations de contexte enregistrées dans PowerShell ne sont pas mises à jour : il peut donc être nécessaire de les effacer ou de changer des paramètres. Si vous transférez l’abonnement à un compte dans un autre locataire et que vous décidez de déplacer l’abonnement vers ce locataire, tous les utilisateurs, groupes et principaux de service qui disposaient d’[attributions de rôles Azure](../../role-based-access-control/role-assignments-portal.md) pour accéder aux ressources de l’abonnement perdent cet accès. Il peut en résulter un temps d’arrêt du service.

## <a name="new-account-usage-and-billing-history"></a>Utilisation de nouveau compte et historique de facturation

La seule information accessible aux utilisateurs du nouveau compte est le coût du dernier mois de votre abonnement. Le reste de l’historique d’utilisation et de facturation n’est pas transféré avec l’abonnement.

## <a name="manually-migrate-data-and-services"></a>Migrer manuellement des données et services

Lorsque vous transférez un abonnement, ses ressources l’accompagnent. Si vous ne pouvez pas transférer la propriété d’un abonnement, vous pouvez migrer ses ressources manuellement. Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

## <a name="remaining-subscription-credits"></a>Crédits d’abonnement restants 

Si vous avez un abonnement Visual Studio ou Microsoft Partner Network, vous bénéficiez de crédits mensuels. Votre crédit n’est pas reporté avec l’abonnement dans le nouveau compte. L’utilisateur qui a accepté la demande de transfert doit avoir une licence Visual Studio pour procéder à l’acceptation. L’abonnement utilise le crédit Visual Studio disponible dans le compte de l’utilisateur. Pour plus d’informations, consultez [Transfert des abonnements Visual Studio et Partner Network](../manage/billing-subscription-transfer.md#transfer-visual-studio-and-partner-network-subscriptions).

## <a name="users-keep-access-to-transferred-resources"></a>Les utilisateurs conservent l’accès aux ressources transférées

Gardez à l’esprit que les utilisateurs ayant accès aux ressources d’un abonnement conservent cet accès lors du transfert de la propriété. Cependant, les [rôles d’administrateur](../manage/add-change-subscription-administrator.md) et les [attributions de rôles Azure](../../role-based-access-control/role-assignments-portal.md) peuvent être supprimées. La perte de l’accès se produit quand votre compte se trouve dans un locataire Azure AD autre que le locataire de l’abonnement et que l’utilisateur qui a envoyé la demande de transfert déplace l’abonnement vers le locataire de votre compte. 

Vous pouvez afficher les utilisateurs disposant d’attributions de rôle Azure pour accéder aux ressources de l’abonnement dans le portail Azure. Accédez à la [page Abonnement du portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). Choisissez ensuite l’abonnement que vous souhaitez vérifier, puis sélectionnez **Contrôle d’accès (IAM)** dans le volet gauche. Sélectionnez **Attributions de rôles** en haut de la page. La page des attributions de rôles liste tous les utilisateurs qui disposent d’un accès à l’abonnement.

Même si les [attributions de rôles Azure](../../role-based-access-control/role-assignments-portal.md) sont supprimées pendant le transfert, les utilisateurs du compte du propriétaire d’origine peuvent continuer à avoir accès à l’abonnement par le biais d’autres mécanismes de sécurité, notamment :

* Certificats de gestion accordant à l’utilisateur des droits d’administrateur sur les ressources d’abonnement. Pour plus d'informations, consultez la rubrique [Créer et télécharger un certificat de gestion pour Microsoft Azure](../../cloud-services/cloud-services-certs-create.md).
* Touches d’accès rapide pour les services tels que Storage. Pour plus d’informations, consultez la rubrique [À propos des comptes de stockage Azure](../../storage/common/storage-account-create.md).
* Informations d’identification d’accès à distance pour les services tels que les machines virtuelles Azure.

Si le destinataire doit restreindre l’accès aux ressources, il doit envisager la mise à jour des secrets associés au service. La plupart des ressources peuvent être mises à jour. Connectez-vous au [portail Azure](https://portal.azure.com) puis, dans le menu Hub, sélectionnez **Toutes les ressources**. Ensuite, sélectionnez la ressource. Dans la page de la ressource, sélectionnez **Paramètres**. Ici, vous pouvez afficher et mettre à jour des secrets existants.

## <a name="you-pay-for-usage-when-you-receive-ownership"></a>Vous payez pour l’utilisation quand vous recevez la propriété

Votre compte est responsable du paiement de toute utilisation qui est signalée à compter du moment du transfert. Certaines opérations peuvent avoir été effectuées avant le transfert mais déclarées ultérieurement. L’utilisation figure sur la facture de votre compte.

## <a name="use-a-different-payment-method"></a>Utiliser un autre mode de paiement

Lors de l’acceptation de la demande de transfert, vous pouvez sélectionner un mode de paiement existant lié à votre compte, ou ajouter un nouveau mode de paiement.

## <a name="transfer-enterprise-agreement-subscription-ownership"></a>Transférer la propriété d’un abonnement Contrat Entreprise

L’Administrateur d’entreprise peut mettre à jour la propriété de tout compte, même si le propriétaire original du compte ne fait plus partie de l’organisation. Pour plus d’informations sur le transfert de comptes Contrat Entreprise Azure, consultez [Transferts Azure Enterprise](../manage/ea-transfers.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Transférer la propriété de facturation d’un abonnement Azure](../manage/billing-subscription-transfer.md)
