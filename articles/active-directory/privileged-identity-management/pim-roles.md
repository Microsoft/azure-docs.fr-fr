---
title: Rôles que vous ne pouvez pas gérer dans Privileged Identity Management - Azure Active Directory | Microsoft Docs
description: Décrit les rôles que vous ne pouvez pas gérer dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 05/11/2020
ms.author: curtand
ms.custom: pim ; H1Hack27Feb2017;oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: c72587c5486ed61215fd20c215a1dd194f4b7bc4
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92372410"
---
# <a name="roles-you-cant-manage-in-privileged-identity-management"></a>Rôles que vous ne pouvez pas gérer dans Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) vous permet de gérer tous les [rôles Azure AD](../roles/permissions-reference.md) et tous les [rôles Azure](../../role-based-access-control/built-in-roles.md). Les rôles Azure peuvent également inclure vos rôles personnalisés, attachés à vos groupes d’administration, abonnements, groupes de ressources et ressources. Toutefois, il y a quelques rôles que vous ne pouvez pas gérer dans PIM. Cet article décrit les rôles que vous ne pouvez pas gérer dans Privileged Identity Management (PIM).

## <a name="classic-subscription-administrator-roles"></a>Rôles d’administrateur d’abonnements classique

Vous ne pouvez pas gérer les rôles d’administrateur d’abonnements classiques suivants dans Privileged Identity Management :

- Administrateur de comptes
- Administrateur de services
- Coadministrateur

Pour plus d’informations sur les rôles d’administrateur d’abonnements classiques, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

## <a name="what-about-microsoft-365-admin-roles"></a>Qu’en est-il des rôles d’administrateur Microsoft 365 ?

Nous prenons en charge tous les rôles Microsoft 365 dans l’expérience de portail Rôles et administrateurs Azure AD, tels que l’administrateur Exchange et l’administrateur SharePoint, mais nous ne prenons pas en charge les rôles spécifiques dans les sessions RBAC Exchange ou RBAC SharePoint. Pour plus d’informations sur ces services Microsoft 365, consultez [Rôles d’administrateur Microsoft 365](/office365/admin/add-users/about-admin-roles).

> [!NOTE]
> Utilisateurs éligibles pour le rôle d’administrateur SharePoint, le rôle d’administrateur de périphériques, ainsi que tous les rôles tentant d’accéder au Centre de sécurité et de conformité Microsoft peuvent rencontrer des retards pouvant atteindre plusieurs heures après l’activation de leur rôle. Nous travaillons en collaboration avec ces équipes pour résoudre les problèmes.

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles dans Azure AD dans Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Attribuer des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-assign-roles.md)