---
title: Afficher le rapport d’audit pour les rôles de ressources Azure dans Privileged Identity Management (PIM) – Azure AD | Microsoft Docs
description: Affichez l’activité et l’historique d’audit des rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 01/10/2020
ms.author: curtand
ms.reviewer: shaunliu
ms.collection: M365-identity-device-management
ms.openlocfilehash: 45144f64789a19390984c3f9f6a660e3c3300215
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002491"
---
# <a name="view-activity-and-audit-history-for-azure-resource-roles-in-privileged-identity-management"></a>Affichez l’activité et l’historique d’audit des rôles de ressources Azure dans Privileged Identity Management

Avec Azure Active Directory (Azure AD) Privileged Identity Management (PIM), vous pouvez voir l’activité, les activations et l’historique d’audit des rôles de ressources Azure au sein de votre organisation. Cela inclut les abonnements, les groupes de ressources et même les machines virtuelles. Toutes les ressources du Portail Azure qui utilisent la fonctionnalité de contrôle d’accès en fonction du rôle d’Azure peuvent tirer parti des capacités de gestion de la sécurité et des cycles de vie proposées par Privileged Identity Management.

> [!NOTE]
> Si votre organisation possède des fonctions de gestion externalisées pour un fournisseur de services qui utilise la [gestion des ressources déléguées Azure](../../lighthouse/concepts/azure-delegated-resource-management.md), les attributions de rôles autorisées par ce fournisseur de services ne seront pas affichées ici.

## <a name="view-activity-and-activations"></a>Afficher l’activité et les activations

Pour connaître les actions qu’un utilisateur en particulier a effectuées dans différentes ressources, vous pouvez voir l’activité des ressources Azure qui est associée à une période d’activation donnée.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource pour laquelle vous souhaitez voir l’activité et les activations.

1. Sélectionnez **Rôles** ou **Membres**.

1. Sélectionnez un utilisateur.

    Vous voyez un résumé des actions de l’utilisateur dans les ressources Azure par date. Il montre également les dernières activations de rôle sur cette même période.

    ![Détails de l’utilisateur avec résumé des activités des ressources et activations de rôle](media/azure-pim-resource-rbac/rbac-user-details.png)

1. Sélectionnez une activation de rôle spécifique pour voir les détails et l’activité des ressources Azure correspondante survenue lorsque l’utilisateur était actif.

    [![Role activation selected and activity details](media/azure-pim-resource-rbac/export-membership.png "Activation de rôle sélectionnée et détails de l’activité")](media/azure-pim-resource-rbac/export-membership.png)

## <a name="export-role-assignments-with-children"></a>Exporter les attributions de rôle ayant des enfants

Vous pouvez avoir une exigence de conformité selon laquelle vous devez fournir une liste complète des attributions de rôle à des auditeurs. Privileged Identity Management permet d’interroger les attributions de rôle sur une ressource particulière, ce qui inclut les attributions de rôle de toutes les ressources enfants. Avant, il était difficile pour les administrateurs d’obtenir une liste complète des attributions de rôle d’un abonnement, car ils devaient les exporter pour chaque ressource spécifique. Avec Privileged Identity Management, vous pouvez interroger toutes les attributions de rôle actives et éligibles d’un abonnement, notamment les attributions de rôle des ressources et de tous les groupes de ressources.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource pour laquelle vous souhaitez exporter des attributions de rôle, par exemple un abonnement.

1. Sélectionnez **Membres**.

1. Sélectionnez **Exporter** pour ouvrir le volet Exporter l’appartenance.

    [![Export membership pane to export all members](media/azure-pim-resource-rbac/export-membership.png "Page Exporter l’appartenance pour exporter tous les membres")](media/azure-pim-resource-rbac/export-membership.png)

1. Sélectionnez **Exporter tous les membres** pour exporter toutes les attributions de rôle dans un fichier CSV.

    ![Attributions de rôle exportées dans le fichier CSV comme affiché dans Excel](media/azure-pim-resource-rbac/export-csv.png)

## <a name="view-resource-audit-history"></a>Afficher l’historique d’audit des ressources

L’audit des ressources vous donne une vue d’ensemble de l’activité des rôles d’une ressource.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource pour laquelle vous souhaitez consulter l’historique d’audit.

1. Sélectionnez un **audit de ressources**.

1. Filtrez l’historique en choisissant une date prédéfinie ou une plage personnalisée.

    [![Resource audit list with filters](media/azure-pim-resource-rbac/rbac-resource-audit.png "Liste des audits de ressources avec filtres")](media/azure-pim-resource-rbac/rbac-resource-audit.png)

1. Pour **Type d’audit**, sélectionnez **Activer (Attribué + Activé)** .

    [![Resource audit list filtered by Activate audit type](media/azure-pim-resource-rbac/rbac-audit-activity.png "Liste des audits de ressources filtrée par Activé")](media/azure-pim-resource-rbac/rbac-audit-activity.png) ![Resource audit list that is filtered by Activate audit type](media/azure-pim-resource-rbac/rbac-audit-activity.png)

1. Sous **Action**, cliquez sur **(activité)** pour un utilisateur, afin d’afficher le détail de l’activité de cet utilisateur dans les ressources Azure.

    ![Détails de l’activité utilisateur pour une action particulière](media/azure-pim-resource-rbac/rbac-audit-activity-details.png)

## <a name="view-my-audit"></a>Afficher mon audit

Mon audit vous permet d’afficher votre activité personnelle de rôle.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource pour laquelle vous souhaitez consulter l’historique d’audit.

1. Sélectionnez **Mon audit**.

1. Filtrez l’historique en choisissant une date prédéfinie ou une plage personnalisée.

    [![Audit list for the current user](media/azure-pim-resource-rbac/my-audit-time.png "Liste d’audits pour l’utilisateur actuel")](media/azure-pim-resource-rbac/my-audit-time.png)

> [!NOTE]
> Pour accéder à l’historique d’audit, vous devez avoir un rôle Administrateur général ou Administrateur de rôle privilégié.

## <a name="get-reason-approver-and-ticket-number-for-approval-events"></a>Obtenir la raison, l’approbateur et le numéro de ticket pour les événements d’approbation

1. Connectez-vous au [portail Azure](https://aad.portal.azure.com) avec les autorisations du rôle Administrateur de rôle privilégié, puis ouvrez Azure AD.
1. Sélectionnez **Journaux d’audit**.
1. Utilisez le filtre **Service** pour afficher uniquement les événements d’audit qui concernent le service Privileged Identity Management. Sur la page **Journaux d’audit**, vous pouvez :

    - Consulter la raison d’un événement d’audit dans la colonne **Motif du statut**.
    - Consulter l’approbateur dans la colonne **Initié par (acteur)** pour l’événement « Demande d’ajout de membre au rôle approuvée ».

    [![Filter the audit log for the PIM service](media/azure-pim-resource-rbac/filter-audit-logs.png "Filtrer le journal d’audit pour le service PIM")](media/azure-pim-resource-rbac/filter-audit-logs.png)

1. Sélectionnez un événement de journal d’audit pour afficher le numéro de ticket sous l’onglet **Activité** du volet **Détails**.
  
    [![Check the ticket number for the audit event](media/azure-pim-resource-rbac/audit-event-ticket-number.png "Vérifier le numéro de ticket pour l’événement d’audit")](media/azure-pim-resource-rbac/audit-event-ticket-number.png)]

1. Vous pouvez afficher le demandeur (personne qui active le rôle) sous l’onglet **Cibles** du volet **Détails** d’un événement d’audit. Il existe trois types de cibles pour les rôles de ressources Azure :

    - Le rôle (**Type** = Rôle)
    - Le demandeur (**Type** = Autre)
    - L’approbateur (**Type** = Utilisateur)

    [![Check the target type](media/azure-pim-resource-rbac/audit-event-target-type.png "Vérifier le type de cible")](media/azure-pim-resource-rbac/audit-event-target-type.png)

En règle générale, l’événement de journal situé juste au-dessus de l’événement d’approbation est un événement « Ajout de membre au rôle terminé » où **Initié par (acteur)** est le demandeur. Dans la plupart des cas, vous n’aurez pas besoin de rechercher le demandeur dans la demande d’approbation du point de vue de l’audit.

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-assign-roles.md)
- [Approuver ou rejeter des demandes de rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-approval-workflow.md)
- [Afficher l’activité et l’historique d’audit des rôles Azure AD dans Privileged Identity Management](pim-how-to-use-audit-log.md)
