---
title: Afficher le rapport d’audit pour les affectations de groupe d’accès privilégié dans Privileged Identity Management (PIM) – Azure AD | Microsoft Docs
description: Affichez l’activité et l’historique d’audit des affectations de groupe d’accès privilégié dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: article
ms.workload: identity
ms.subservice: pim
ms.date: 07/27/2020
ms.author: curtand
ms.reviewer: shaunliu
ms.collection: M365-identity-device-management
ms.openlocfilehash: 94dd5c2579eb7ce96dea70c6354df8ec84125bd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88141667"
---
# <a name="audit-activity-history-for-privileged-access-group-assignments-preview-in-privileged-identity-management"></a>Historique des activités d’audit pour les affectations de groupe d’accès privilégié (préversion) dans Privileged Identity Management

Grâce à Privileged Identity Management (PIM), vous pouvez voir l’activité, les activations et l’historique d’audit des membres et des propriétaires de groupes d’accès privilégié Azure au sein de votre organisation Azure Active Directory (Azure AD).

> [!NOTE]
> Si votre organisation possède des fonctions de gestion externalisées pour un fournisseur de services qui utilise la [gestion des ressources déléguées Azure](../../lighthouse/concepts/azure-delegated-resource-management.md), les attributions de rôles autorisées par ce fournisseur de services ne seront pas affichées ici.

Procédez comme suit pour afficher l’historique d’audit des groupes d’accès privilégié.

## <a name="view-resource-audit-history"></a>Afficher l’historique d’audit des ressources

L’**audit des ressources** vous donne un aperçu de toutes les activités associées à vos groupes d’accès privilégié.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Groupes d’accès privilégiés (Préversion)** .

1. Sélectionnez le groupe d’accès privilégié pour lequel vous souhaitez consulter l’historique d’audit.

1. Sous **Activité**, sélectionnez **Audit des ressources**.

1. Filtrez l’historique en choisissant une date prédéfinie ou une plage personnalisée.

    ![Liste des audits de ressources avec filtres](media/groups-audit/groups-resource-audit.png)

## <a name="view-my-audit"></a>Afficher mon audit

**Mon audit** vous permet d’afficher votre activité personnelle de rôle pour un groupe d’accès privilégié.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Groupes d’accès privilégiés (Préversion)** .

1. Sélectionnez le groupe d’accès privilégié pour lequel vous souhaitez consulter l’historique d’audit.

1. Sous **Activité**, sélectionnez **Mon audit**.

1. Filtrez l’historique en choisissant une date prédéfinie ou une plage personnalisée.

    ![Liste d’audits pour l’utilisateur actuel](media/azure-pim-resource-rbac/my-audit-time.png)

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer l’appartenance à un groupe et la propriété d’un groupe dans Privileged Identity Management](groups-assign-member-owner.md)
- [Approuver ou refuser des demandes de groupes d’accès privilégié dans PIM](groups-approval-workflow.md)
- [Afficher l'historique d'audit pour les rôles Azure AD dans PIM](groups-audit.md)
