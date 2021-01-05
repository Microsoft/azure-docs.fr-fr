---
title: Configurer des identités managées sur une machine virtuelle Azure en utilisant PowerShell - Azure AD
description: Instructions pas à pas pour configurer des identités managées pour les ressources Azure sur une machine virtuelle Azure en utilisant PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/19/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4b4209ce159c9d0bbee01dd422b98832f6bb5713
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90968995"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-powershell"></a>Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Les identités managées pour ressources Azure fournissent automatiquement aux services Azure une identité managée dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code.

Cet article explique comment effectuer les opérations suivantes d’identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md). **Veillez à consulter la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour exécuter les exemples de scripts, vous avez deux options :
    - Utiliser [Azure Cloud Shell](../../cloud-shell/overview.md), que vous pouvez ouvrir en utilisant le bouton **Essayer** dans le coin supérieur droit des blocs de code.
    - Exécuter les scripts localement en installant la dernière version d’[Azure PowerShell](/powershell/azure/install-az-ps), puis en vous connectant à Azure en utilisant `Connect-AzAccount`. 

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Dans cette section, vous allez découvrir comment activer et désactiver l’identité managée affectée par le système en utilisant Azure PowerShell.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Activer une identité managée affectée par le système lors de la création d’une machine virtuelle Azure

Pour créer une machine virtuelle Azure avec l’identité managée affectée par le système sur une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Consultez l’un des démarrages rapides de machine virtuelle Azure suivants, en ne complétant que les sections nécessaires (« Se connecter à Azure », « Créer un groupe de ressources », « Créer un groupe de mise en réseau », « Créer la machine virtuelle »).

    Lorsque vous accédez à la section « Créer la machine virtuelle », apportez une légère modification à la syntaxe de l’applet de commande [New-AzVMConfig](/powershell/module/az.compute/new-azvm). Veillez à ajouter un paramètre `-IdentityType SystemAssigned` afin d’approvisionner la machine virtuelle avec l’identité affectée par le système activée, par exemple :

    ```azurepowershell-interactive
    $vmConfig = New-AzVMConfig -VMName myVM -IdentityType SystemAssigned ...
    ```

   - [Créer une machine virtuelle Windows avec PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
   - [Créer une machine virtuelle Linux avec PowerShell](../../virtual-machines/linux/quick-create-powershell.md)

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-vm"></a>Activer une identité managée affectée par le système sur une machine virtuelle Azure existante

Pour activer l’identité managée affectée par le système sur une machine virtuelle qui en était dépourvue initialement, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Récupérez les propriétés de la machine virtuelle à l’aide de l’applet de commande `Get-AzVM`. Ensuite, pour activer une identité managée affectée par le système, utilisez le commutateur `-IdentityType` avec l’applet de commande [Update-AzVM](/powershell/module/az.compute/update-azvm) :

   ```azurepowershell-interactive
   $vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -IdentityType SystemAssigned
   ```

### <a name="add-vm-system-assigned-identity-to-a-group"></a>Ajoutez l’identité attribuée par le système d’une machine virtuelle à un groupe

Après avoir activé l'identité attribuée par le système sur une machine virtuelle, vous pouvez l’ajouter à un groupe.  La procédure suivante permet d'ajouter une identité attribuée par le système à un groupe.

1. Récupérez et notez le `ObjectID` (comme spécifié dans le champ `Id` des valeurs renvoyées) du principal de service de la machine virtuelle :

   ```azurepowershell-interactive
   Get-AzADServicePrincipal -displayname "myVM"
   ```

1. Récupérez et notez le `ObjectID` (comme spécifié dans le champ `Id` des valeurs renvoyées) du groupe :

   ```azurepowershell-interactive
   Get-AzADGroup -searchstring "myGroup"
   ```

1. Ajouter le principal de service de la machine virtuelle au groupe :

   ```azurepowershell-interactive
   Add-AzureADGroupMember -ObjectId "<objectID of group>" -RefObjectId "<object id of VM service principal>"
   ```

## <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>Désactiver une identité managée affectée par le système d’une machine virtuelle Azure

Pour désactiver l’identité managée affectée par le système sur une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

Si vous disposez d’une machine virtuelle qui n’a plus besoin de l’identité managée affectée par le système, mais qui a toujours besoin d’identités managées affectées par l’utilisateur, utilisez l’applet de commande suivante :

1. Récupérez les propriétés de la machine virtuelle à l’aide de l’applet de commande `Get-AzVM` et définissez le paramètre `-IdentityType` sur `UserAssigned` :

   ```azurepowershell-interactive
   $vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType "UserAssigned"
   ```

Si vous disposez d’une machine virtuelle qui n’a plus besoin de l’identité managée affectée par le système et qui n’a pas d’identité managée affectée par l’utilisateur, utilisez les commandes suivantes :

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```

## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Dans cette section, vous allez découvrir comment ajouter et supprimer une identité managée affectée par l’utilisateur sur une machine virtuelle en utilisant Azure PowerShell.

### <a name="assign-a-user-assigned-managed-identity-to-a-vm-during-creation"></a>Attribuer une identité managée affectée par l’utilisateur à une machine virtuelle lors de sa création

Pour affecter une identité managée affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) et [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Consultez l’un des démarrages rapides de machine virtuelle Azure suivants, en ne complétant que les sections nécessaires (« Se connecter à Azure », « Créer un groupe de ressources », « Créer un groupe de mise en réseau », « Créer la machine virtuelle »).

    Quand vous parvenez à la section « Créer la machine virtuelle », apportez une légère modification à la syntaxe de l’applet de commande [`New-AzVMConfig`](/powershell/module/az.compute/new-azvm). Ajoutez les paramètres `-IdentityType UserAssigned` et `-IdentityID` pour approvisionner la machine virtuelle avec une identité affectée par l’utilisateur.  Remplacez `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>` et `<USER ASSIGNED IDENTITY NAME>` par vos propres valeurs.  Par exemple :

    ```azurepowershell-interactive
    $vmConfig = New-AzVMConfig -VMName <VM NAME> -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>..."
    ```

    - [Créer une machine virtuelle Windows avec PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
    - [Créer une machine virtuelle Linux avec PowerShell](../../virtual-machines/linux/quick-create-powershell.md)


### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Attribuer une identité managée affectée par l’utilisateur à une machine virtuelle Azure existante

Pour affecter une identité managée affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) et [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Créez une identité managée affectée par l’utilisateur en utilisant l’applet de commande [New-AzUserAssignedIdentity](/powershell/module/az.managedserviceidentity/new-azuserassignedidentity).  Notez la valeur `Id` dans la sortie, car vous en aurez besoin à l’étape suivante.

   > [!IMPORTANT]
   > La création d’identités managées affectées par l’utilisateur prend uniquement en charge les caractères alphanumériques, les traits de soulignement et les traits d’union (0-9, a-z, A-Z, \_ ou -). De plus, le nom doit être limité entre 3 et 128 caractères pour que l’attribution à la machine virtuelle/au groupe de machines virtuelles identiques (VMSS) fonctionne correctement. Pour plus d’informations, consultez [FAQ et problèmes connus](known-issues.md)

   ```azurepowershell-interactive
   New-AzUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
   ```
1. Récupérez les propriétés de la machine virtuelle à l’aide de l’applet de commande `Get-AzVM`. Ensuite, pour attribuer une identité managée affectée par l’utilisateur à la machine virtuelle Azure, utilisez les commutateurs `-IdentityType` et `-IdentityID` avec l’applet de commande [Update-AzVM](/powershell/module/az.compute/update-azvm).  La valeur du paramètre `-IdentityId` est la valeur `Id` que vous avez notée à l’étape précédente.  Remplacez `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>` et `<USER ASSIGNED IDENTITY NAME>` par vos propres valeurs.

   > [!WARNING]
   > Pour conserver les identités managées précédemment affectées par l’utilisateur à la machine virtuelle, effectuez une requête sur la propriété `Identity` de l’objet machine virtuelle (par exemple, `$vm.Identity`).  Si de telles identités managées sont retournées, ajoutez-les à la commande suivante, avec la nouvelle identité managée affectée par l’utilisateur que vous souhaitez affecter à la machine virtuelle.

   ```azurepowershell-interactive
   $vm = Get-AzVM -ResourceGroupName <RESOURCE GROUP> -Name <VM NAME>
   Update-AzVM -ResourceGroupName <RESOURCE GROUP> -VM $vm -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>"
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Supprimer une identité managée affectée par l’utilisateur d’une machine virtuelle Azure

Pour supprimer une identité affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

Si votre machine virtuelle a plusieurs identités managées affectées par l’utilisateur, vous pouvez les supprimer toutes sauf la dernière à l’aide des commandes suivantes. N’oubliez pas de remplacer les valeurs des paramètres `<RESOURCE GROUP>` et `<VM NAME>` par vos propres valeurs. `<USER ASSIGNED IDENTITY NAME>` est la propriété de nom de l’identité managée affectée par l’utilisateur, qui doit rester sur la machine virtuelle. Pour obtenir cette information, effectuez une requête sur la propriété `Identity` de l’objet machine virtuelle.  Par exemple, `$vm.Identity`:

```azurepowershell-interactive
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType UserAssigned -IdentityID <USER ASSIGNED IDENTITY NAME>
```
Si votre machine virtuelle ne dispose pas d’une identité managée affectée par le système, et si vous souhaitez supprimer toutes les identités managées affectées par l’utilisateur, utilisez la commande suivante :

```azurepowershell-interactive
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```
Si votre machine virtuelle dispose d’identités managées affectées tant par le système que par l’utilisateur, vous pouvez supprimer toutes les identités managées affectées par l’utilisateur en choisissant de n’utiliser que l’identité managée affectée par le système.

```azurepowershell-interactive
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des identités managées pour les ressources Azure](overview.md)
- Pour obtenir les guides de démarrages rapides complets sur la création de machines virtuelles Azure, consultez :

  - [Créer une machine virtuelle Windows avec PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
  - [Créer une machine virtuelle Linux avec PowerShell](../../virtual-machines/linux/quick-create-powershell.md)
