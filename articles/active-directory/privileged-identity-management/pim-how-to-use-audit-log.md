---
title: Afficher le rapport de journal d’audit pour les rôles Azure AD dans Azure AD PIM | Microsoft Docs
description: Découvrez comment afficher l’historique du journal d’audit des rôles Azure AD dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 01/07/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8481c562ecbab1f26e877e55a5a9454695ddf4c0
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370829"
---
# <a name="view-audit-history-for-azure-ad-roles-in-privileged-identity-management"></a>Afficher l’historique d’audit des rôles Azure AD dans Privileged Identity Management

Vous pouvez utiliser l’historique d’audit PIM (Privileged Identity Management) pour voir toutes les activations et toutes les affectations de rôles qui ont été effectuées au cours des 30 derniers jours pour tous les rôles privilégiés. Si vous souhaitez consulter l’historique d’audit complet de l’activité dans votre organisation Azure Active Directory (Azure AD), notamment l’activité de l’administrateur, de l’utilisateur final et de la synchronisation, vous pouvez utiliser les [rapports d’activité et de sécurité d’Azure Active Directory](../reports-monitoring/overview-reports.md).

## <a name="determine-your-version-of-pim"></a>Déterminer votre version de PIM

Depuis novembre 2019, la partie Rôles Azure AD de Privileged Identity Management est mise à jour vers une nouvelle version qui correspond à l’expérience des rôles de ressources Azure. Cela permet de créer des fonctionnalités supplémentaires et d’apporter des [modifications à l’API existante](azure-ad-roles-features.md#api-changes). Pendant que la nouvelle version est déployée, les procédures que vous suivez dans cet article dépendent de la version de Privileged Identity Management que vous possédez actuellement. Pour déterminer la version de Privileged Identity Management dont vous disposez, procédez de la manière décrite dans cette section. Une fois que vous connaissez votre version de Privileged Identity Management, vous pouvez sélectionner les procédures de cet article qui correspondent à cette version.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un utilisateur avec le rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator).
1. Ouvrez **Azure AD Privileged Identity Management** . Si une bannière figure en haut de la page de présentation, suivez les instructions sous l’onglet **Nouvelle version** de cet article. Sinon, suivez les instructions sous l’onglet **Version précédente** .

    [![Capture d’écran montrant la page « Rôles Azure AD – Historique d’audit des rôles d’annuaire ».](media/pim-how-to-use-audit-log/directory-roles-audit-history.png "Sélectionner l’onglet correspondant à votre version")](media/pim-how-to-use-audit-log/directory-roles-audit-history.png)

# <a name="new-version"></a>[Nouvelle version](#tab/new)

Suivez ces étapes pour afficher l’historique des audits pour les rôles Azure AD.

## <a name="view-resource-audit-history"></a>Afficher l’historique d’audit des ressources

L’audit des ressources vous donne un aperçu de toutes les activités associées à vos rôles Azure AD.

1. Ouvrez **Azure AD Privileged Identity Management** .

1. Sélectionnez des **rôles Azure AD** .

1. Sélectionnez un **audit de ressources** .

1. Filtrez l’historique en choisissant une date prédéfinie ou une plage personnalisée.

    ![Liste des audits de ressources avec filtres](media/azure-pim-resource-rbac/rbac-resource-audit.png)

## <a name="view-my-audit"></a>Afficher mon audit

Mon audit vous permet d’afficher votre activité personnelle de rôle.

1. Ouvrez **Azure AD Privileged Identity Management** .

1. Sélectionnez des **rôles Azure AD** .

1. Sélectionnez la ressource pour laquelle vous souhaitez consulter l’historique d’audit.

1. Sélectionnez **Mon audit** .

1. Filtrez l’historique en choisissant une date prédéfinie ou une plage personnalisée.

    ![Liste d’audits pour l’utilisateur actuel](media/azure-pim-resource-rbac/my-audit-time.png)

# <a name="previous-version"></a>[Version précédente](#tab/previous)

## <a name="view-audit-history"></a>Afficher l’historique d'audit

Suivez ces étapes pour afficher l’historique des audits pour les rôles Azure AD.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un nom d’utilisateur qui est membre du rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator).

1. Ouvrez **Azure AD Privileged Identity Management** .

1. Sélectionnez des **rôles Azure AD** .

1. Sélectionnez l’ **historique d’audit des rôles d’annuaire** .

    En fonction de l’historique d’audit, un histogramme s’affiche dans lequel figurent le nombre total d’activations, le nombre maximal d’activations par jour et la moyenne d’activations par jour.

    [![Nouvelle version des rôles Azure AD](media/pim-how-to-use-audit-log/directory-roles-audit-history.png "Afficher l’historique d’audit des rôles d’annuaire")](media/pim-how-to-use-audit-log/directory-roles-audit-history.png)

    Au bas de la page, un tableau s’affiche avec des informations sur chaque action de l’historique d’audit disponible. Voici la signification de ces colonnes :

    | Colonne | Description |
    | --- | --- |
    | Temps | Moment auquel une action s’est produite. |
    | Demandeur | Personne qui a demandé l’activation d’un rôle ou sa modification. Si la valeur est **Système Azure** , consultez l’historique d’audit Azure pour obtenir plus d’informations. |
    | Action | Actions effectuées par le demandeur. Il peut s’agir des actions Affecter, Annuler l’affectation, Activer, Désactiver ou AddedOutsidePIM. |
    | Membre | Utilisateur qui active un rôle ou est affecté à ce rôle. |
    | Role | Rôle affecté ou activé par l’utilisateur. |
    | Raisonnement | Texte entré dans le champ Motif pendant l’activation. |
    | Expiration | Date d’expiration d’un rôle activé. S’applique uniquement aux affectations de rôles éligibles. |

1. Pour trier l’historique d’audit, utilisez les boutons **Date** , **Action** et **Rôle** .

## <a name="filter-audit-history"></a>Filtrer l’historique d’audit

1. En haut de la page de l’historique d’audit, cliquez sur le bouton **Filtrer** .

    Le volet **Mettre à jour les paramètres du graphique** s’affiche.

1. Dans **Intervalle de temps** , sélectionnez un intervalle de temps.

1. Dans **Rôles** , activez les cases à cocher pour indiquer les rôles que vous souhaitez afficher.

    ![Volet Mettre à jour les paramètres du graphique](media/pim-how-to-use-audit-log/update-chart-parameters.png)

1. Sélectionnez **Terminé** pour voir l’historique d’audit filtré.

## <a name="get-reason-approver-and-ticket-number-for-approval-events"></a>Obtenir la raison, l’approbateur et le numéro de ticket pour les événements d’approbation

1. Connectez-vous au [portail Azure](https://aad.portal.azure.com) avec les autorisations du rôle Administrateur de rôle privilégié, puis ouvrez Azure AD.
1. Sélectionnez **Journaux d’audit** .
1. Utilisez le filtre **Service** pour afficher uniquement les événements d’audit qui concernent le service Privileged Identity Management. Sur la page **Journaux d’audit** , vous pouvez :

    - Consulter la raison d’un événement d’audit dans la colonne **Motif du statut** .
    - Consulter l’approbateur dans la colonne **Initié par (acteur)** pour l’événement « Demande d’ajout de membre au rôle approuvée ».

    [![Capture d’écran montrant la page « Journaux d’audit » avec le menu « Initié par (intervenant) » ouvert et « PIM » sélectionné.](media/pim-how-to-use-audit-log/filter-audit-logs.png "Filtrer le journal d’audit pour le service PIM")](media/pim-how-to-use-audit-log/filter-audit-logs.png)

1. Sélectionnez un événement de journal d’audit pour afficher le numéro de ticket sous l’onglet **Activité** du volet **Détails** .
  
    [![Capture d’écran montrant la page « Journaux d’audit » avec le numéro de ticket mis en évidence dans le volet « Détails ».](media/pim-how-to-use-audit-log/audit-event-ticket-number.png "Vérifier le numéro de ticket pour l’événement d’audit")](media/pim-how-to-use-audit-log/audit-event-ticket-number.png)

1. Vous pouvez afficher le demandeur (personne qui active le rôle) sous l’onglet **Cibles** du volet **Détails** d’un événement d’audit. Il existe deux types de cibles pour les rôles Azure AD :

    - Le rôle ( **Type** = Rôle)
    - Le demandeur ( **Type** = Utilisateur)

En règle générale, l’événement de journal d’audit situé juste au-dessus de l’événement d’approbation est un événement « Ajout de membre au rôle terminé » où **Initié par (acteur)** est le demandeur. Dans la plupart des cas, vous n’aurez pas besoin de rechercher le demandeur dans la demande d’approbation du point de vue de l’audit.

---

## <a name="next-steps"></a>Étapes suivantes

- [Afficher l’activité et l’historique d’audit des rôles de ressources Azure dans Privileged Identity Management](azure-pim-resource-rbac.md)
