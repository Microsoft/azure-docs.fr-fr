---
title: Transférer des abonnements Azure entre des abonnés et des CSP
description: Découvrez comment transférer des abonnements Azure entre des abonnés et des CSP.
author: bandersmsft
ms.reviewer: dhgandhi
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 11/17/2020
ms.author: banders
ms.openlocfilehash: 4cc213db19a567f18c5ff483b64a75e3bbeef892
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962108"
---
# <a name="transfer-azure-subscriptions-between-subscribers-and-csps"></a>Transférer des abonnements Azure entre des abonnés et des CSP

Cet article présente les étapes principales utilisées pour transférer des abonnements Azure vers et à partir des partenaires fournisseurs de solutions cloud (CSP) et leurs clients. Les informations fournies ici sont destinées à l’abonné Azure pour l’aider à se coordonner avec son partenaire. Les informations que les partenaires Microsoft utilisent pour le processus de transfert figurent dans la section [Découvrez comment transférer les abonnements Azure d’un client à un autre partenaire](/partner-center/switch-azure-subscriptions-to-a-different-partner).

Avant de commencer une demande de transfert, vous devez télécharger ou exporter toutes les informations de facturation et de coût que vous voulez conserver. Les informations de facturation et d’utilisation ne sont pas transférées avec l’abonnement. Pour plus d’informations sur l’exportation des données de gestion des coûts, consultez [Créer et gérer des données exportées](../costs/tutorial-export-acm-data.md). Pour plus d’informations sur le téléchargement de vos données de facturation et d’utilisation, consultez [Télécharger ou afficher votre facture Azure et vos données d’utilisation quotidienne](download-azure-invoice-daily-usage-date.md).

Si vous avez des réservations existantes, celles-ci cessent de s’appliquer après le transfert d’un abonnement. Veillez à [annuler les réservations et à les rembourser](../reservations/exchange-and-refund-azure-reservations.md) avant de transférer un abonnement.

## <a name="transfer-ea-subscriptions-to-a-csp-partner"></a>Transférer des abonnements EA à un partenaire CSP

Les partenaires CSP de facturation directe certifiés [Fournisseur de services managés Azure Expert (MSP)](https://partner.microsoft.com/membership/azure-expert-msp) peuvent demander le transfert d’abonnements Azure pour leurs clients ayant contracté un contrat Entreprise direct (EA). Les transferts d’abonnements sont autorisés uniquement pour les clients qui ont accepté un Contrat client Microsoft (MCA) et acheté un plan Azure avec le programme CSP.

Lorsque la demande est approuvée, le CSP peut alors fournir une facture combinée à ses clients. Pour en savoir plus sur le transfert d’abonnements par les CSP, consultez [Obtenir la propriété de facturation des abonnements Azure pour votre compte MPA](mpa-request-ownership.md).

>[!IMPORTANT]
> Après le transfert d’un abonnement Contrat Entreprise à un partenaire CSP, toute augmentation du quota précédemment appliquée à l’abonnement Contrat Entreprise sera rétablie à sa valeur par défaut. Si un quota supplémentaire est nécessaire après le transfert de l’abonnement, demandez à votre fournisseur CSP de soumettre une demande d’[augmentation de quota](../../azure-portal/supportability/regional-quota-requests.md). 

## <a name="other-subscription-transfers-to-a-csp-partner"></a>Autres transferts d’abonnements à un partenaire CSP

Pour transférer tout autre abonnement Azure à un partenaire CSP, l’abonné doit déplacer les ressources des abonnements sources vers les abonnements CSP. Respectez les instructions suivantes pour déplacer des ressources entre des abonnements.

1. Collaborez avec votre partenaire CSP pour créer des abonnements Azure CSP cibles.
1. Vérifiez que les abonnements CSP sources et cibles sont dans le même locataire Azure Active Directory (Azure AD).  
    Vous ne pouvez pas changer le locataire Azure AD d’un abonnement Azure CSP. Au lieu de cela, vous devez ajouter ou associer l’abonnement source au locataire CSP Azure AD. Pour plus d’informations, consultez [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).
    > [!IMPORTANT]
    > - Lorsque vous associez un abonnement à un autre annuaire Azure AD, les utilisateurs possédant des rôles attribués à l’aide du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md) perdent leur accès. Les administrateurs d’abonnements classiques, entre autres les administrateurs de services et les coadministrateurs perdent également leur accès.
    > - Les attributions de stratégie sont également supprimées d’un abonnement lorsque celui-ci est associé à un autre répertoire.
1. Le compte d’utilisateur que vous utilisez pour effectuer le transfert doit disposer d’un accès propriétaire de [RBAC Azure](add-change-subscription-administrator.md) aux deux abonnements.
1. Avant de commencer, [validez](/rest/api/resources/resources/validatemoveresources) le fait que toutes les ressources Azure peuvent être déplacées de l’abonnement source à l’abonnement de destination.  
    Certaines ressources Azure ne peuvent pas être déplacées entre des abonnements. Pour afficher la liste complète des ressources Azure qui peuvent être déplacées, consultez [Prise en charge des opérations de déplacement pour les ressources](../../azure-resource-manager/management/move-support-resources.md).
    > [!IMPORTANT]
    >  - Azure CSP prend en charge uniquement les ressources Azure Resource Manager. Si des ressources Azure dans l’abonnement source ont été créées à l’aide du modèle de déploiement Azure classique, vous devez les migrer vers [Azure Resource Manager](/azure/cloud-solution-provider/migration/ea-payg-to-azure-csp/ea-open-direct-asm-to-arm) avant la migration. Vous devez être partenaire pour pouvoir afficher la page web.

1. Vérifiez que tous les services d’abonnement sources utilisent le modèle Azure Resource Manager. Ensuite, transférez les ressources de l’abonnement source vers l’abonnement de destination à l’aide du [déplacement de ressources Azure](../../azure-resource-manager/management/move-resource-group-and-subscription.md).
    > [!IMPORTANT]
    >  - Le déplacement des ressources Azure entre des abonnements peut entraîner un temps d’arrêt du service, en fonction des ressources des abonnements.

## <a name="transfer-csp-subscription-to-other-offer"></a>Transférer un abonnement CSP à une autre offre

Pour transférer tout autre abonnement d’un partenaire CSP vers une autre offre Azure, l’abonné doit déplacer des ressources entre des abonnements CSP sources et des abonnements CSP cibles.

1. Créer des abonnements Azure cibles.
1. Vérifiez que les abonnements source et cible sont dans le même locataire Azure Active Directory (Azure AD). Pour plus d’informations sur la modification d’un locataire Azure AD, consultez [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).
    Notez que l’option de changement d’annuaire n’est pas prise en charge pour l’abonnement CSP. Par exemple, vous passez d’un abonnement CSP à un abonnement avec paiement à l’utilisation. Vous devez changer l’annuaire de l’abonnement Paiement à l’utilisation pour qu’il corresponde à l’annuaire CSP.

    > [!IMPORTANT]
    >  - Lorsque vous associez un abonnement à un autre annuaire, les utilisateurs auxquels des rôles ont été attribués à l’aide du [RBAC Azure](../../role-based-access-control/role-assignments-portal.md) perdent leur accès. Les administrateurs d’abonnements classiques, entre autres les administrateurs de services et les coadministrateurs perdent également leur accès.
    >  - Les attributions de stratégie sont également supprimées d’un abonnement lorsque celui-ci est associé à un autre répertoire.

1. Le compte d’utilisateur que vous utilisez pour effectuer le transfert doit disposer d’un accès propriétaire de [RBAC Azure](add-change-subscription-administrator.md) aux deux abonnements.
1. Avant de commencer, [validez](/rest/api/resources/resources/validatemoveresources) le fait que toutes les ressources Azure peuvent être déplacées de l’abonnement source à l’abonnement de destination.
    > [!IMPORTANT]
    >  - Certaines ressources Azure ne peuvent pas être déplacées entre des abonnements. Pour afficher la liste complète des ressources Azure qui peuvent être déplacées, consultez [Prise en charge des opérations de déplacement pour les ressources](../../azure-resource-manager/management/move-support-resources.md).

1. Transférez les ressources de l’abonnement source vers l’abonnement de destination à l’aide du [déplacement de ressources Azure](../../azure-resource-manager/management/move-resource-group-and-subscription.md).
    > [!IMPORTANT]
    >  - Le déplacement des ressources Azure entre des abonnements peut entraîner un temps d’arrêt du service, en fonction des ressources des abonnements.

## <a name="next-steps"></a>Étapes suivantes
- [Obtenir la propriété de facturation des abonnements Azure associés pour votre compte MPA](mpa-request-ownership.md).
- En savoir plus sur la façon de [Gérer des comptes et des abonnements avec la facturation Azure](../index.yml).
