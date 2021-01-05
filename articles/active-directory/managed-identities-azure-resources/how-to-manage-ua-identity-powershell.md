---
title: Créer, répertorier et supprimer une identité managée affectée par l’utilisateur en utilisant Azure PowerShell – Azure AD
description: Instructions pas à pas pour créer, lister et supprimer une identité managée affectée par l’utilisateur en utilisant Azure PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/02/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 636f40f0c425c25dfe7f41f1f404afc90ed5ba56
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548220"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-azure-powershell"></a>Créer, lister ou supprimer une identité managée affectée par l’utilisateur en utilisant Azure PowerShell

Les identités managées pour ressources Azure fournissent aux services Azure une identité managée dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier auprès de services prenant en charge l’authentification Azure AD, sans avoir recours à des informations d’identification dans votre code. 

Dans cet article, vous allez découvrir comment créer, lister et supprimer une identité managée affectée par l’utilisateur en utilisant Azure PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md). **Veillez à consulter la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour exécuter les exemples de scripts, vous avez deux options :
    - Utiliser [Azure Cloud Shell](../../cloud-shell/overview.md), que vous pouvez ouvrir à l’aide du bouton **Essayer** dans le coin supérieur droit des blocs de code.
    - Exécutez les scripts localement avec Azure PowerShell, comme décrit dans la section suivante.

### <a name="configure-azure-powershell-locally"></a>Configurer Azure PowerShell localement

Pour utiliser Azure PowerShell localement dans cet article (au lieu d’utiliser Cloud Shell), effectuez les étapes suivantes :

1. Installez [la dernière version d’Azure PowerShell](/powershell/azure/install-az-ps) si ce n’est déjà fait.

1. Connectez-vous à Azure :

    ```azurepowershell
    Connect-AzAccount
    ```

1. Installez la [dernière version de PowerShellGet](/powershell/scripting/gallery/installing-psget#for-systems-with-powershell-50-or-newer-you-can-install-the-latest-powershellget).

    ```azurepowershell
    Install-Module -Name PowerShellGet -AllowPrerelease
    ```

    Vous devrez peut-être `Exit` de la session PowerShell en cours après avoir exécuté cette commande, pour l’étape suivante.

1. Installez la version préliminaire du module `Az.ManagedServiceIdentity`, afin d’effectuer les opérations d’identité managée affectée par l’utilisateur qui sont décrites dans cet article :

    ```azurepowershell
    Install-Module -Name Az.ManagedServiceIdentity -AllowPrerelease
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur

Pour créer une identité managée affectée par l’utilisateur, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Pour créer une identité managée affectée par l’utilisateur, utilisez la commande `New-AzUserAssignedIdentity`. Le paramètre `ResourceGroupName` spécifie le groupe de ressources dans lequel créer l’identité managée affectée par l’utilisateur, et le paramètre `-Name` spécifie son nom. Remplacez les valeurs des paramètres `<RESOURCE GROUP>` et `<USER ASSIGNED IDENTITY NAME>` par vos propres valeurs :

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

 ```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>Répertorier les identités managées affectées par l’utilisateur

Pour créer une identité managée affectée par l’utilisateur, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator) ou [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Pour répertorier les identités affectées par l’utilisateur, utilisez la commande [Get-AzUserAssigned].  Le paramètre `-ResourceGroupName` spécifie le groupe de ressources dans lequel l’identité managée affectée par l’utilisateur a été créée. Remplacez `<RESOURCE GROUP>` par votre propre valeur :

```azurepowershell-interactive
Get-AzUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP>
```
Dans la réponse, les identités managées affectées par l’utilisateur ont une valeur `"Microsoft.ManagedIdentity/userAssignedIdentities"` retournée pour la clé, `Type`.

`Type :Microsoft.ManagedIdentity/userAssignedIdentities`

## <a name="delete-a-user-assigned-managed-identity"></a>Supprimer une identité managée affectée par l’utilisateur

Pour supprimer une identité managée affectée par l’utilisateur, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Pour supprimer une identité managée affectée par l’utilisateur, utilisez la commande `Remove-AzUserAssignedIdentity`.  Le paramètre `-ResourceGroupName` spécifie le groupe de ressources dans lequel l’identité managée affectée par l’utilisateur a été créée, et le paramètre `-Name` spécifie son nom. Remplacez les valeurs des paramètres `<RESOURCE GROUP>` et `<USER ASSIGNED IDENTITY NAME>` par vos propres valeurs :

 ```azurepowershell-interactive
Remove-AzUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP> -Name <USER ASSIGNED IDENTITY NAME>
```
> [!NOTE]
> La suppression d’une identité managée affectée par l’utilisateur n’a pas pour effet de supprimer la référence d’une ressource à laquelle elle a été affectée. Les attributions d’identités doivent être supprimées séparément.

## <a name="next-steps"></a>Étapes suivantes

Pour la liste complète et les détails des commandes d’identités managées Azure PowerShell pour ressources Azure, consultez [Az.ManagedServiceIdentity](/powershell/module/az.managedserviceidentity#managed_service_identity).
