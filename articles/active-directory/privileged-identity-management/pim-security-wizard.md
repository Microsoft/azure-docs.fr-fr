---
title: Rôles Azure AD - Détection et insights (préversion) (anciennement Assistant Sécurité) dans Privileged Identity Management - Azure Active Directory
description: La fonctionnalité Détection et insights (anciennement Assistant Sécurité) vous aide à convertir des attributions de rôles Azure AD permanentes en attributions juste-à-temps avec Privileged Identity Management.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 09/01/2020
ms.author: curtand
ms.custom: pim ; H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4662e9fb537a93cb87c554e319256e2eca40d2e7
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92372427"
---
# <a name="discovery-and-insights-preview-for-azure-ad-roles-formerly-security-wizard"></a>Détection et insights (préversion) pour les rôles Azure AD (anciennement Assistant Sécurité)

Si vous commencez à utiliser Privileged Identity Management (PIM) dans votre organisation Azure Active Directory (Azure AD), vous pouvez utiliser la page **Détection et insights (préversion)** pour vous lancer. Cette fonctionnalité vous indique qui est affecté aux rôles privilégiés dans votre organisation et comment utiliser PIM pour modifier rapidement les attributions de rôles permanentes en attributions juste-à-temps. Vous pouvez afficher ou modifier vos attributions de rôles privilégiés permanentes dans **Détection et insights (préversion)** . Il s’agit d’un outil d’analyse et d’un outil d’action.

## <a name="discovery-and-insights-preview"></a>Détection et insights (préversion)

Avant que votre organisation commence à utiliser Privileged Identity Management, toutes les attributions de rôles sont permanentes. Les utilisateurs se trouvent toujours dans les rôles qui leur sont attribués, même s’ils n’ont pas besoin des privilèges associés. Détection et insights (préversion), qui remplace l’ancien Assistant Sécurité, affiche une liste des rôles privilégiés et le nombre d’utilisateurs actuellement dans ces rôles. Vous pouvez répertorier les attributions d’un rôle pour en savoir plus sur les utilisateurs affectés si un ou plusieurs d’entre eux ne vous sont pas familiers.

:heavy_check_mark: **Microsoft recommande** de conserver deux comptes de secours qui sont attribués de façon permanente au rôle d’administrateur général. Assurez-vous que ces comptes ne nécessitent pas le même mécanisme d’authentification multifacteur que vos comptes d’administration normaux pour se connecter, comme décrit dans la section [ Gérer les comptes d’accès d’urgence dans Azure AD](../roles/security-emergency-access.md).

En outre, conservez les attributions de rôles permanentes si un utilisateur possède un compte Microsoft (en d’autres termes, un compte pour se connecter aux services Microsoft tels que Skype ou Outlook.com). Si vous avez besoin de l’authentification multifacteur pour un utilisateur disposant d’un compte Microsoft afin d’activer une attribution de rôle, l’utilisateur est verrouillé.

## <a name="open-discovery-and-insights-preview"></a>Ouvrir Détection et insights (préversion)

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management** .

1. Sélectionnez **Rôles Azure AD** , puis sélectionnez **Détection et insights (préversion)** . L’ouverture de la page lance le processus de détection pour rechercher les attributions de rôles pertinentes.

    ![Rôles Azure AD - Page Détection et insights présentant les 3 options](./media/pim-security-wizard/new-preview-link.png)

1. Sélectionnez **Réduire les administrateurs généraux** .

    ![Capture d’écran représentant « Détection et insights (préversion) » avec l’action « Réduire les administrateurs généraux » sélectionnée.](./media/pim-security-wizard/new-preview-page.png)

1. Passez en revue la liste des attributions du rôle Administrateur général.

    ![Réduire les administrateurs généraux - Volet Rôle présentant tous les Administrateurs généraux](./media/pim-security-wizard/new-global-administrator-list.png)

1. Sélectionnez **Suivant** pour sélectionner les utilisateurs ou les groupes que vous souhaitez rendre éligibles, puis sélectionnez **Rendre éligible** ou **Supprimer l’attribution** .

    ![Convertir les membres dans la page éligible avec les options permettant de sélectionner les membres que vous souhaitez rendre éligibles pour les rôles](./media/pim-security-wizard/new-global-administrator-buttons.png)

1. Vous pouvez également demander à tous les administrateurs généraux de réviser leur propre accès.

    ![Page Administrateurs généraux, section Révisions d’accès](./media/pim-security-wizard/new-global-administrator-access-review.png)

1. Après avoir sélectionné l’une de ces modifications, vous verrez une notification Azure.

1. Vous pouvez ensuite sélectionner **Éliminer l’accès permanent** ou **Vérifier les principaux de service** pour répéter les étapes ci-dessus sur d’autres rôles privilégiés et sur les attributions de rôles de principal de service. Pour les attributions de rôles de principal de service, vous ne pouvez supprimer que les attributions de rôles.

    ![Options Insights supplémentaires pour éliminer l’accès permanent et vérifier les principaux de service ](./media/pim-security-wizard/new-preview-page-service-principals.png)

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles dans Azure AD dans Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Accorder l’accès à d’autres administrateurs pour la gestion de Privileged Identity Management](pim-how-to-give-access-to-pim.md)
