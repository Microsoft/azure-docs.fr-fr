---
title: Autorisations dans Azure Sentinel | Microsoft Docs
description: Cet article explique comment Azure Sentinel utilise le contrôle d'accès en fonction du rôle Azure pour attribuer des autorisations à des utilisateurs, et il identifie les actions autorisées pour chaque rôle.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/28/2020
ms.author: yelevin
ms.openlocfilehash: 9c100ca82b751a050e74cce85e1794701a789288
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95498467"
---
# <a name="permissions-in-azure-sentinel"></a>Autorisations dans Azure Sentinel

Azure Sentinel utilise le [contrôle d'accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) pour fournir des [rôles intégrés](../role-based-access-control/built-in-roles.md) susceptibles d'être attribués à des utilisateurs, des groupes et des services dans Azure.

Utilisez Azure RBAC pour créer et attribuer des rôles au sein de votre équipe en charge des opérations de sécurité afin d'accorder l'accès approprié à Azure Sentinel. Les différents rôles vous donnent un contrôle précis sur ce que les utilisateurs d’Azure Sentinel peuvent voir et faire. Les rôles Azure peuvent être attribués directement dans l’espace de travail Azure Sentinel (voir la remarque ci-dessous), ou dans un abonnement ou un groupe de ressources auquel appartient l’espace de travail, dont Azure Sentinel héritera.

## <a name="roles-for-working-in-azure-sentinel"></a>Rôles pour travailler dans Azure Sentinel

### <a name="azure-sentinel-specific-roles"></a>Rôles spécifiques à Azure Sentinel

Il existe trois rôles Azure Sentinel intégrés dédiés.

**Tous les rôles intégrés Azure Sentinel accordent un accès en lecture aux données de votre espace de travail Azure Sentinel.**

- [Lecteur Azure Sentinel](../role-based-access-control/built-in-roles.md#azure-sentinel-reader) : il peut visualiser les données, les incidents, les classeurs et d’autres ressources Azure Sentinel.

- [Répondeur Azure Sentinel](../role-based-access-control/built-in-roles.md#azure-sentinel-responder) : il peut, en plus des éléments ci-dessus, gérer les incidents (affecter, ignorer, etc.)

- [Contributeur Azure Sentinel](../role-based-access-control/built-in-roles.md#azure-sentinel-contributor) : il peut, en plus des éléments ci-dessus, créer et modifier des classeurs, des règles d’analytique et d’autres ressources Azure Sentinel.

> [!NOTE]
>
> - Pour de meilleurs résultats, ces rôles doivent être attribués sur le **groupe de ressources** contenant l’espace de travail Azure Sentinel. De cette façon, les rôles s’appliquent à toutes les ressources déployées pour prendre en charge Azure Sentinel, car ces ressources doivent également être placées dans ce même groupe de ressources.
>
> - Une autre option consiste à attribuer les rôles directement sur l’**espace de travail** Azure Sentinel lui-même. Dans ce cas, vous devez également attribuer les mêmes rôles sur la **ressource de solution** SecurityInsights dans cet espace de travail. Il peut être nécessaire de les attribuer également sur d’autres ressources, et vous devrez constamment gérer les attributions de rôles sur les ressources.

### <a name="additional-roles-and-permissions"></a>Rôles et autorisations supplémentaires

Il peut être nécessaire d’affecter des rôles supplémentaires ou des autorisations spécifiques aux utilisateurs ayant des exigences particulières pour accomplir leurs tâches.

- Utilisation de playbooks pour automatiser les réponses aux menaces

    Azure Sentinel utilise des **playbooks** pour la réponse automatisée aux menaces. Les playbooks sont basés sur **Azure Logic Apps** et constituent une ressource Azure distincte. Vous pouvez attribuer à des membres spécifiques de votre équipe en charge des opérations de sécurité la possibilité d’utiliser Logic Apps pour les opérations SOAR (Security Orchestration, Automation, and Response). Vous pouvez utiliser le rôle [Contributeur d’application logique](../role-based-access-control/built-in-roles.md#logic-app-contributor) ou le rôle [Opérateur d’application logique](../role-based-access-control/built-in-roles.md#logic-app-operator) pour attribuer une autorisation explicite pour l’utilisation des playbooks.

- Connexion de sources de données à Azure Sentinel

    Pour qu’un utilisateur ajoute des **connecteurs de données**, vous devez attribuer les autorisations d’accès en écriture à l’utilisateur sur l’espace de travail Azure Sentinel. Notez également les autorisations supplémentaires nécessaires pour chaque connecteur, comme indiqué dans la page du connecteur approprié.

- Utilisateurs invités attribuant des incidents

    Si un utilisateur invité doit être en mesure d’attribuer des incidents, il doit se voir attribuer le rôle [Lecteur de répertoire](../active-directory/roles/permissions-reference.md#directory-readers) en plus du rôle Répondeur Azure Sentinel. Notez que ce rôle *n'est pas* un rôle Azure, mais un rôle **Azure Active Directory**, et qu'il est attribué par défaut aux utilisateurs ordinaires (non invités). 

Pour une comparaison côte à côte, consultez le [tableau ci-dessous](#roles-and-allowed-actions).

### <a name="other-roles-you-might-see-assigned"></a>Autres rôles que vous pouvez voir attribués

En attribuant des rôles Azure spécifiques à Azure Sentinel, vous pouvez rencontrer d’autres rôles Azure et Log Analytics qui peuvent avoir été attribués aux utilisateurs à d’autres fins. Notez que ces rôles accordent un ensemble plus important d’autorisations qui incluent l’accès à votre espace de travail Azure Sentinel et à d’autres ressources :

- **Rôles Azure :** [Propriétaire](../role-based-access-control/built-in-roles.md#owner), [Contributeur](../role-based-access-control/built-in-roles.md#contributor) et [Lecteur](../role-based-access-control/built-in-roles.md#reader). Les rôles Azure accordent l’accès à l’ensemble de vos ressources Azure, notamment aux espaces de travail Log Analytics et aux ressources Azure Sentinel.

- **Rôles Log Analytics :** [Contributeur Log Analytics](../role-based-access-control/built-in-roles.md#log-analytics-contributor) et [Lecteur Log Analytics](../role-based-access-control/built-in-roles.md#log-analytics-reader). Les rôles Log Analytics accordent l’accès à vos espaces de travail Log Analytics. 

Par exemple, un utilisateur auquel est attribué le rôle **Lecteur Azure Sentinel**, mais pas le rôle **Contributeur Azure Sentinel**, pourra quand même modifier des éléments dans Azure Sentinel si le rôle **Contributeur** au niveau d’Azure lui est attribué. Par conséquent, si vous voulez accorder des autorisations à un utilisateur seulement dans Azure Sentinel, vous devez supprimer soigneusement les autorisations antérieures de cet utilisateur, en veillant à ne pas supprimer un accès nécessaire à une autre ressource.

## <a name="roles-and-allowed-actions"></a>Rôles et actions autorisées

Le tableau suivant récapitule les rôles et les actions autorisées dans Azure Sentinel. 

| Role | Créer et exécuter des playbooks| Créer et modifier des classeurs, des règles d’analytique et d’autres ressources Azure Sentinel | Gérer les incidents (ignorer, attribuer, etc.) | Visualiser des données, des incidents, des classeurs et d’autres ressources Azure Sentinel |
|---|---|---|---|---|
| Lecteur Azure Sentinel | -- | -- | -- | &#10003; |
| Répondeur Azure Sentinel | -- | -- | &#10003; | &#10003; |
| Contributeur Azure Sentinel | -- | &#10003; | &#10003; | &#10003; |
| Contributeur Azure Sentinel + contributeur d’application logique | &#10003; | &#10003; | &#10003; | &#10003; |

## <a name="custom-roles-and-advanced-azure-rbac"></a>Rôles personnalisés et RBAC Azure avancé

- En plus d’utiliser des rôles intégrés Azure, ou au lieu d’en utiliser, vous pouvez créer des rôles personnalisés Azure pour Azure Sentinel. Les rôles personnalisés Azure pour Azure Sentinel sont créés de la même façon que les autres [rôles personnalisés Azure](../role-based-access-control/custom-roles-rest.md#create-a-custom-role), c'est-à-dire en fonction des [autorisations spécifiques à Azure Sentinel](../role-based-access-control/resource-provider-operations.md#microsoftsecurityinsights) et aux [ressources Azure Log Analytics](../role-based-access-control/resource-provider-operations.md#microsoftoperationalinsights).

- Vous pouvez utiliser le contrôle d’accès en fonction du rôle Azure avancé Log Analytics sur les données de votre espace de travail Azure Sentinel. Cela comprend à la fois le contrôle Azure RBAC basé sur le type de données et le contrôle Azure RBAC centré sur les ressources. Pour plus d'informations sur les rôles Log Analytics, consultez [Gérer les données du journal et les espaces de travail dans Azure Monitor](../azure-monitor/platform/manage-access.md#manage-access-using-workspace-permissions).

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à utiliser des rôles pour les utilisateurs Azure Sentinel et découvert ce que chaque rôle permet aux utilisateurs de faire.

* [Blog Azure Sentinel](https://aka.ms/azuresentinelblog) : accédez à des billets de blog sur la sécurité et la conformité Azure.
