---
title: Rôles intégrés Windows Virtual Desktop – Azure
description: Vue d’ensemble des rôles intégrés pour Windows Virtual Desktop disponibles pour le RBAC Azure.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 52f0151fd4be7505cf7beea0eeb54f8e34404997
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97577545"
---
# <a name="built-in-roles-for-windows-virtual-desktop"></a>Rôles intégrés pour Windows Virtual Desktop

Windows Virtual Desktop utilise des contrôles d’accès en fonction du rôle (RBAC) Azure pour attribuer des rôles aux utilisateurs et aux administrateurs. Ces rôles donnent aux administrateurs l’autorisation d’effectuer certaines tâches. Pour en savoir plus sur les rôles intégrés pour le RBAC Azure, consultez [Rôles intégrés Azure](../role-based-access-control/built-in-roles.md).

Les rôles intégrés standard pour Azure sont Propriétaire, Contributeur et Lecteur. Toutefois, Windows Virtual Desktop intègre des rôles supplémentaires qui vous permettent de séparer les rôles de gestion pour les pools d’ordinateurs hôtes, les groupes d’applications et les espaces de travail. Cette séparation vous permet d’exercer un contrôle plus précis sur les tâches d’administration. Ces rôles sont nommés conformément aux rôles standard et à la méthodologie du privilège minimum d’Azure.

Windows Virtual Desktop n’a pas de rôle Propriétaire spécifique. Toutefois, vous pouvez utiliser un rôle Propriétaire standard pour les objets service.

## <a name="desktop-virtualization-contributor"></a>Contributeur de virtualisation des services Bureau

Le rôle Contributeur de virtualisation des services bureau vous permet de gérer tous les aspects du déploiement. Toutefois, il ne vous accorde pas l’accès aux ressources de calcul. Vous aurez également besoin du rôle Administrateur de l’accès utilisateur pour publier des groupes d’applications pour des utilisateurs ou groupes d’utilisateurs.


- Microsoft.DesktopVirtualization/\* 
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="desktop-virtualization-reader"></a>Lecteur de virtualisation des services Bureau

Le rôle Lecteur de virtualisation des services Bureau vous permet d’afficher tout le contenu du déploiement, mais pas d’y apporter des modifications.

- Microsoft.DesktopVirtualization/\*/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="host-pool-contributor"></a>Contributeur du pool d’hôtes

Le rôle Contributeur du pool d’hôtes vous permet de gérer tous les aspects des pools hôtes, y compris l’accès aux ressources. Vous aurez besoin d’un rôle contributeur supplémentaire, Contributeur de machine virtuelle, pour créer des machines virtuelles. Vous aurez besoin des rôles Contributeur de groupe d’applications et Contributeur d’espace de travail pour créer un pool d’hôtes à l’aide du portail, ou vous pouvez utiliser le rôle Contributeur de virtualisation des services Bureau.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/hostpools/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="host-pool-reader"></a>Lecteur de pool d’hôtes

Le rôle Lecteur de pool d’hôtes vous permet d’afficher tout le contenu du pool d’hôtes, mais pas d’y apporter des modifications.

- Microsoft.DesktopVirtualization/hostpools/\*/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="application-group-contributor"></a>Contributeur de groupe d’applications

Le rôle Contributeur de groupe d’applications vous permet de gérer tous les aspects des groupes d’applications. Si vous souhaitez publier des groupes d’applications vers des utilisateurs ou groupes d’utilisateurs, vous avez besoin du rôle Administrateur de l’accès utilisateur.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/applicationgroups/\*
- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="application-group-reader"></a>Lecteur de groupe d’applications

Le rôle Lecteur de groupe d’applications vous permet d’afficher tout le contenu du groupe d’applications, mais pas d’y apporter des modifications.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/applicationgroups/\*/read
- Microsoft.DesktopVirtualization/applicationgroups/read
- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="workspace-contributor"></a>Contributeur d’espace de travail

Le rôle Contributeur d’espace de travail vous permet de gérer tous les aspects des espaces de travail. Pour recevoir des informations sur les applications ajoutées aux groupes d’applications, le rôle Lecteur de groupe d’applications doit également vous être attribué.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/workspaces/\*
- Microsoft.DesktopVirtualization/applicationgroups/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="workspace-reader"></a>Lecteur d’espace de travail

Le rôle Lecteur d’espace de travail vous permet d’afficher tout le contenu de l’espace de travail, mais pas d’y apporter des modifications.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/workspaces/read
- Microsoft.DesktopVirtualization/applicationgroups/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="user-session-operator"></a>Opérateur de session utilisateur

Le rôle Opérateur de session utilisateur vous permet d’envoyer des messages, de déconnecter des sessions et d’utiliser la fonction « logoff » pour signer des sessions hors de l’hôte de la session. Toutefois, ce rôle ne vous permet pas de gérer l’hôte de la session, par exemple, de le supprimer, d’en modifier le mode maintenance, et ainsi de suite. Ce rôle peut voir les affectations, mais ne peut pas modifier les administrateurs. Nous vous recommandons d’attribuer ce rôle à des pools d’hôtes spécifiques. Si vous accordez cette autorisation au niveau d’un groupe de ressources, l’administrateur dispose de l’autorisation de lecture sur tous les pools d’hôtes d’un groupe de ressources.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/usersessions/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="session-host-operator"></a>Contributeur d’hôte de session

Le rôle Contributeur d’hôte de session vous permet d’afficher et de supprimer des hôtes de session, ainsi que de modifier le mode maintenance. Vous ne pouvez pas ajouter d’hôtes de session à l’aide du Portail Azure, car vous n’avez pas d’autorisation d’écriture pour des objets pool d’hôtes. Si le jeton d’inscription est valide (généré et non expiré), vous pouvez utiliser ce rôle pour ajouter des hôtes de session au pool d’hôtes en dehors du portail Azure si l’administrateur dispose d’autorisations de calcul via le rôle Contributeur de machine virtuelle.

La liste suivante décrit les autorisations auxquelles ce rôle peut accéder :

- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*
