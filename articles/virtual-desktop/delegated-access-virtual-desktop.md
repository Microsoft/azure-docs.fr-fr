---
title: Accès délégué dans Windows Virtual Desktop - Azure
description: Découvrez comment déléguer des fonctionnalités d’administration lors d'un déploiement de Windows Virtual Desktop, avec exemples.
author: Heidilohr
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f2aa2c74704cf89c082d2837b39e82902efa0a62
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88010053"
---
# <a name="delegated-access-in-windows-virtual-desktop"></a>Accès délégué dans Windows Virtual Desktop

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/delegated-access-virtual-desktop-2019.md).

Windows Virtual Desktop dispose d’un modèle d’accès délégué qui vous permet de définir les droits d’accès d’un utilisateur particulier en lui attribuant un rôle. Une attribution de rôle se compose de trois éléments : un principal de sécurité, une définition de rôle et une étendue. Le modèle d’accès délégué de Windows Virtual Desktop est basé sur le modèle RBAC d’Azure. Pour en savoir plus sur les attributions de rôle spécifique et leurs composants, consultez [la vue d’ensemble du contrôle d'accès en fonction du rôle Azure](../role-based-access-control/built-in-roles.md).

L’accès délégué de Windows Virtual Desktop prend en charge les valeurs suivantes pour chaque élément de l’attribution de rôle :

* Principal de sécurité
    * Utilisateurs
    * Groupes d’utilisateurs
    * Principaux de service
* Définition de rôle
    * Rôles intégrés
    * Rôles personnalisés
* Étendue
    * Pools hôtes
    * Groupes d’applications
    * Workspaces

## <a name="powershell-cmdlets-for-role-assignments"></a>Cmdlets Azure PowerShell pour les attributions de rôles

Avant de commencer, veillez à suivre les instructions figurant dans [Configurer le module PowerShell](powershell-module.md) pour configurer le module PowerShell Windows Virtual Desktop, si ce n’est pas déjà fait.

Windows Virtual Desktop utilise le contrôle d’accès en fonction du rôle (RBAC) Azure pendant la publication de groupes d’applications vers des utilisateurs ou des groupes d’utilisateurs. Le rôle d’utilisateur Virtualisation du Bureau est attribué à l’utilisateur ou au groupe d’utilisateurs et l’étendue correspond au groupe d’applications. Ce rôle donne à l’utilisateur un accès spécial aux données sur le groupe d’applications.

Exécutez la cmdlet suivante pour ajouter des utilisateurs Azure Active Directory à un groupe d’applications :

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Exécutez la cmdlet suivante pour ajouter un groupe d’utilisateurs Azure Active Directory à un groupe d’applications :

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une liste plus complète des cmdlets PowerShell que chaque rôle peut utiliser, consultez les [Informations de référence PowerShell](/powershell/windows-virtual-desktop/overview).

Pour obtenir la liste complète des rôles pris en charge dans RBAC Azure, consultez [Rôles intégrés Azure](../role-based-access-control/built-in-roles.md).

Pour obtenir des instructions et savoir comment configurer un environnement Bureau virtuel Windows, consultez [Environnement dans Windows Virtual Desktop](environment-setup.md).
